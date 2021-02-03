---
layout: post
title: Too many words on S3 object naming & access
date: 2021-02-01 18:32 -0400
tags:
  - AWS
  - S3
---

<small>Reading time: 15 minutes</small>

Amazon S3 is fairly old in cloud years, so it’s not without it historical quirks and dark corners. But when it comes to object naming, things are pretty straightforward: [“You can use any UTF-8 character in an object key name“](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-key-guidelines). This is true, as far as I know, without exception. There are no restricted characters, no magic sequences, and no character is treated differently than any other character.

It’s worth remembering here that S3 doesn’t have a concept of folder hierarchy, and any notion that it does is simply UI or API sugar. Objects are identified entirely by the bucket they belong to and their key name (and sometimes a version ID, but that’s immaterial to this topic). An object that belongs to the `acme-assets` bucket could have a key of `logo.png` or `static/images/logo/200px.png`. In the eyes of S3 there’s nothing intrinsicly different about the object with slashes and the one without. You could just as well have called the second object `static_images_logo_200px.png`. You’d lose out on some UI conveniences, but it’s not the case that one version lives in a folder and the other doesn’t.

You can even name an object with _just_ slashes, if you want. To S3, this object is no different than an object called `README` or `logo.png`. It will behave a little bit strange in the Console, though. But you can try it for yourself:

```python
import boto3
boto3.client('s3').put_object(Key='/////', Body='lorem ipsum', Bucket='my-bucket')
```

```bash
aws s3 ls my-bucket --recursive
# => 2021-02-01 19:07:18         11 /////
```

See? [Nobody cares.](https://www.youtube.com/watch?v=6PeykaclVaA) There’s also nothing special about slashes not being special. Remember: **any UTF-8 character**. `index.html`, `☃️`, `aZ1!-_*'( )?&=+@}{[|^]~<#.mp3`… these are all equally valid object names.

And if we lived inside ~~the matrix~~ S3, this would be the end of the story. We’d simply be able to work directly with any UTF-8 characters we need to, referencing literal object key names as they truly are. But we don’t, and out here in the real world we’re dealing with systems were slashes _do_ matter, where sometimes you can’t use `}` or `☃️` no matter how much you want to, and a litany of other issues that make things more complicated.

Surprisingly more complicated.

The S3 documentation even [includes a guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html#object-key-guidelines) to complications you can encounter will interfacing with S3 object names. These issues aren’t the fault of S3, but with the tools and protocols that interact with S3. Nothing of what comes next refutes the **any UTF-8 character** law of S3.

Consider a couple of the cautionary notices from that guide:

>If an object key name ends with a single period (.), or two periods (..), you can’t download the object using the Amazon S3 console.

> Avoid the following characters in a key name because of significant special handling for consistency across all applications.  [Including:] Left curly brace (“{”)

As you’d hopefully expect by now, if you finesse the APIs and SDKs enough, you can of course do all the things it’s telling you not to. Sometimes you need to get creative with getting files in or out of S3, it all depends on the particulars of the environment. With the Boto3 Python library, no finessing is needed to create objects for these two cases.

```python
boto3.client('s3').put_object(Key='It was the blurst of times...', Body='lorem ipsum', Bucket='my-bucket')
boto3.client('s3').put_object(Key='..', Body='lorem ipsum', Bucket='my-bucket')
boto3.client('s3').put_object(Key='{', Body='lorem ipsum', Bucket='my-bucket')
```

One of the most common uses of S3 will be serving files over HTTP. This could be done with a publicly accessible bucket (maybe don’t), or via CloudFront. Either way, something will be making an HTTP GET request for an object in S3.

Imagine we have taken any steps necessary to create an object called `abominable/☃️` in the `acme-assets` bucket. There are a few bits of information we can use to talk about that object.

The object **key**: `abominable/☃️`

The object’s **ARN**: `arn:aws:s3:::acme-assets/abominable/☃️`

The object’s **S3 URI**: `s3://acme-assets/abominable/☃️`

The object’s **ETag**: `80a751fde577028640c419000e33eba6`

Important things to note: The object key has no leading slash, but the key does contain a slash somewhere.

So far this all should track with our expectations. These values exist within the confines of S3, so there’s no issues with emoji or any other non-ASCII characters. But we’re interested in making an HTTP request for the object, so we have to start to consider any limitations that protocol and tooling may introduce to the equation.

The (modern) way that S3 supports GET requests is with an address like `https://my-bucket.s3.us-east-2.amazonaws.com/my-object-key`. The **path part** of a URL supports a [limited set of characters](https://stackoverflow.com/a/4669755) which, sadly, doesn’t include our snowman.

Taking a URL like `https://example.com/foo/bar.baz`, the path part is `/foo/bar.baz` (note the leading slash), with `foo` and `bar.baz` being considered **path segments** (the bits between slashes). The limited set of characters referenced above refers to the path segments, meaning the complete set of characters that can be used in a path without being encoded includes the slashes between the segments as well. Any characters that need to be represented in a URL path which don’t belong to the slash-including set must use [percent-encoding](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding) (e.g., `@` becomes `%40`, a space becomes `%20`).

While the spec allows for URLs without a path, when making requests for objects in S3 there will always be a not-empty path in the URL. This may not be true when taking actions against S3 buckets themselves, but for S3 object requests, no path would mean no object. And when there is a URL path, it always starts with a slash.

So to generalize, we are going to make an HTTP request to the `my-bucket.s3.us-east-2.amazonaws.com` domain, referencing the object we want in the path, like `/my-object-key`. The key for our object is `abominable/☃️`. We know we can’t use the snowman emoji, and we’re missing a leading slash. We can use [percent-encoding](https://meyerweb.com/eric/tools/dencoder/) to solve the first problem, with `☃️` becoming `%E2%98%83%EF%B8%8F`. And we can add a leading slash to our object key to construct a path, giving us an HTTP URL of `https://acme-assets.s3.us-east-2.amazonaws.com/abominable/%E2%98%83%EF%B8%8F`.

Let’s test if that works:

```bash
curl -I https://acme-assets.s3.us-east-2.amazonaws.com/abominable/%E2%98%83%EF%B8%8F
# => HTTP/1.1 200 OK
# => ETag: "80a751fde577028640c419000e33eba6"
# => Server: AmazonS3
```

We can verify from the ETag that in the response that S3 did in fact find the object we were expecting, so that all worked great. It’s looking like all we have to do to request objects over HTTP is add a leading slash, and percent-encode any characters that aren’t `/`, `A–Z`, `a–z`, `0–9`, `-`, `.`, `_`, `~`, `!`, `$`, `&`, `'`, `(`, `)`, `*`, `+`, `,`, `;`, `=`, `:`, `@`. That would always give us valid URLs, and allow us to reference any object with any UTF-8 characters in the key.

Let’s try a few examples just to confirm.

Here’s an object named `/`. A slash is one of the allowed characters in a URL path (no need to encode), and we add a leading slash to create the URL path. All good. (Console will get a little confused by this, treating it as both a folder and an object in the folder.)

```python
boto3.client('s3').put_object(Key='/', Body='lorem ipsum', Bucket='acme-assets')
```

```bash
curl -I https://acme-assets.s3.us-east-2.amazonaws.com//
# => HTTP/1.1 200 OK
```

Here’s an object whose name is a space. Add a slash, encode the unallowed character, done.

```python
boto3.client('s3').put_object(Key=' ', Body='lorem ipsum', Bucket='acme-assets')
```

```bash
curl -I https://acme-assets.s3.us-east-2.amazonaws.com/%20
# => HTTP/1.1 200 OK
```

How about an object named with every character allowed in URL paths?

```python
boto3.client('s3').put_object(Key="AZaz09-._~!$&'()*+,;=:@", Body='lorem ipsum', Bucket='acme-assets')
```

```bash
curl -I "https://acme-assets.s3.us-east-2.amazonaws.com/AZaz09-._~!$&'()*+,;=:@"
# => HTTP/1.1 403 Forbidden
```

**Uh oh**. We can see the `AZaz09-._~!$&'()*+,;=:@` object in S3, and followed all the rules to construct the HTTP URL, but something’s not happy. The **Object URL** in Console for this object is `https://acme-assets.s3.us-east-2.amazonaws.com/AZaz09-._~!%24%26'()*%2B%2C%3B%3D%3A%40`, which is perhaps a little unexpected. AWS is percent-encoding some characters that should be safe to use in a URL path.

This reveals two things. First, not all allowed URL path characters are acceptable for making HTTP requests to S3. Second is that you can always encode any object key character in a path and S3 will handle them properly.

To demonstrate that second point, consider an S3 object called `Test/abc.mp3`, which contains no objectionable characters. Each of the following requests will return that object.

- `curl https://acme-assets.s3.us-east-2.amazonaws.com/Test/abc.mp3`
- `curl https://acme-assets.s3.us-east-2.amazonaws.com/Test/%61%62%63.mp3`
- `curl https://acme-assets.s3.us-east-2.amazonaws.com/%54est/abc%2Emp3`
- `curl https://acme-assets.s3.us-east-2.amazonaws.com/Test/%61%62%63%2E%6D%70%33`
- `curl https://acme-assets.s3.us-east-2.amazonaws.com/%54%65%73%74%2F%61%62%63%2E%6D%70%33`

It is generally not recommended to encode characters that are safe to use anywhere in a URL, which includes fewer characters than the path segment character set (only `A-Z`, `a-z`, `0-9`, `-`, `_`, `.`, `~`), but S3 allows you to do it when making HTTP requests. Also note that, as always, there’s nothing special about `/` in these object keys; they can be encoded just like everything else. The leading slash on the URL path cannot be encoded, but remember that exists for the purpose of URL structure, and is not part of the object key represented _within_ the path.

Let’s go back and inspect that Object URL from the console. `$`, `&`, `+`, `,`, `;`, `=`, `:`, and `@` were encoded, even though the URL spec says they should have been okay. As we just learned, the Object URL _could_ have encoded every character in the object key, but it chose to only encode these characters. It didn’t even encode everything outside the core set of URL safe characters (`/[\w\-.~/]/`), for example it left `(` and `)` in there. Curious.

If we push on this a bit…

```python
boto3.client('s3').put_object(Key='$', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key='&', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key='+', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key=',', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key=';', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key='=', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key=':', Body='lorem ipsum', Bucket='acme-assets')
boto3.client('s3').put_object(Key='@', Body='lorem ipsum', Bucket='acme-assets')
```

…and request each of those objects, the only one that fails is `+`. The [docs](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html) don’t say anything about special handing of `+` or any of these other characters, but clearly something is different. Let’s see if the URL that failed ealier works if we percent-encode only the `+`: `https://acme-assets.s3.us-east-2.amazonaws.com/AZaz09-._~!$&'()*%2B,;=:@`.

**Yes!** In the common HTTP clients I’ve tried (curl, browsers, Paw, Postman, several libraries, etc), that request worked. That means this isn’t an issue with the outgoing request getting mangled or anything like that, but with S3's parsing of that incoming HTTP request path. It’s unclear why the S3 Console encodes the rest of those characters more aggresively. I suspect there are edge cases where certain _clients_ introduce issues when handling those characters, and it’s just being extra cautious. But there does seem to be some real issue with how the S3 HTTP endpoint handled requests including `+`, so to some extent this is an S3 service problem, but it’s not a fundamental limitation on object key naming. And since most API and SDK calls go through the HTTP endpoints, this is an important thing to remember.

While we’re talking about `+`, you may have some experience using a `+` as a method of encoding a space in some URLs. Without getting into the rules and reasons for when that’s (maybe) allowed, it’s **never** an option for encoding spaces in the path part of a URL according to the spec. So for our needs, making a request for an object called `bagel day` would use a path like `/bagel%20day`, and can’t use `/bagel+day`.

Or can’t we… Since the `+` seems to be the one edge case that exists when making S3 GET requests, and there are some other places on the web where a `+` does encode a space, it’s worthy of an experiment.

```python
boto3.client('s3').put_object(Key='bagel day', Body='lorem ipsum', Bucket='acme-assets')
```

```bash
curl -I https://acme-assets.s3.us-east-2.amazonaws.com/bagel+day
# => HTTP/1.1 200 OK
```

Well that’s interesting. And it explains why the `+` is the only otherwise-acceptable URL path character that needs to be escaped. S3 is treating it as an encoded space in a path, even though that’s non-standard behavior. I would recommend **never** doing that, since you don’t know when your URLs will pass through a tool that wouldn’t expect that and might mangle your URL. Things _should_ be allowed to encode a `+` in a path to `%2B` at any time without issue, but with an S3 HTTP URL that’s using a `+` as a space, doing that encoding would break the request.

To put it another way: most HTTP servers would (should) treat `/bagel+day` and `/bagel%2Bday` as requests for the same resource (words with a plus in between), and `/bagel%20day` as a different resource (words with a space in between). S3 HTTP endpoints, on the other hand, treat `/bagel+day` and `/bagel%20day` as the same (space in between) and `/bagel%2Bday` as different (plus in between). Just like with `☃️`, the **only** way to convey a `+` in an object key to S3 in HTTP URLs is by percent-encoding it. Unlike `☃️`, it is _possible_ to include `+` in an S3 URL unencoded, it just won’t mean what we want.

Now we know a lot about making HTTP GET requests directly to S3 for an object. By and large, the few rules we’ve covered should allow you to get any object out of S3 over HTTP, no matter what it’s named. But how about when working with S3 SDKs? How about when decided on URLs to present to a user in the wild? There are too many languages, libraries, parsers, etc to cover every possible scenario, but here are some examples to give you a sense of how the above rules come into play, and some additional gotchas you should be expecting.

Let’s look at the [AWS SDK for Javascript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html) (verison 2). The [`copyObject`](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#copyObject-property) method, for example, copies an existing object to a new location. The `Key` parameter, the name of the new copy, takes the literal UTF-8 object name, like for `☃️` you’d use `{ Key: '☃️' }`. Pretty simple. (And percent-encoded characters **will not** be decoded.) The `CopySource` parameter references the object to copy, and includes the bucket _and_ object key in the form `my-bucket/my-object-key`. Don’t get this confused with constructing HTTP URL paths – they may look similar but are doing two different things. The docs also say _“The value must be URL encoded”_.

Let’s unpack that last bit some more. We have a value of the form `my-bucket/my-object-key`, which is using a `/` as the delimiter between a bucket name and and object key, which we know can also include slashes. The docs are saying the entire value, not just the object key, needs to be URL encoded. What kind of URL encoding, though? Like for a standard URL path? Like for the special case paths we talked about earlier that encode `+`? Like `encodeURI`? Like `encodeURIComponent`? There’s a lot of options. And don’t forget that `copySource` allows for copying specific object versions. When adding `?versionId=12345` to the value, `?` is a scary character, should that be encoded?

Let’s throw some things at the wall and see what sticks. For all of these examples we’ll be create a copy called `🆕`, and both the old and new copies will live in `acme-assets`. You can assume that the source objects exist. (Also worth noting that some examples in the docs use a `CopySource` in the form `/my-bucket/my-object-key`. The leading slash seems entirely optional.)

#### Basic alphanumeric:

```javascript
// Source object key: simple-name-01
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/simple-name-01' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Alphanumeric with slashes:

```javascript
// Source object key: simple/name/01
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/simple/name/01' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Alphanumeric with encoded object key slashes:

```javascript
// Source object key: simple/name/01
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/simple%2Fname%2F01' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Alphanumeric with encoded delimiter:

```javascript
// Source object key: simple-name-01
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets%2Fsimple-name-01' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Unencoded emoji:

```javascript
// Source object key: ☃️
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/☃️' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👎 This is expected to fail, and does with an `Invalid character in header content` error.

#### Percent-encoded emoji:

```javascript
// Source object key: ☃️
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/%E2%98%83%EF%B8%8F' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Unencoded space:

```javascript
// Source object key: bagel day
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/bagel day' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍 This works, surprisingly

#### Percent-encoded space:

```javascript
// Source object key: bagel day
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/bagel%20day' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Plus-encoded space:

```javascript
// Source object key: bagel day
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/bagel+day' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍 Not too surprising that this works, based on what we saw earlier with HTTP URLs, but non-standard as far URL encoding goes

#### Unencoded non-alphanumeric path-safe characters

```javascript
// Source object key: fancy:name
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/fancy:name' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

#### Unencoded path-unsafe characters:

```javascript
// Source object key: scary#name
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/scary#name' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍 This works, surprisingly

#### More unencoded path-unsafe characters:

```javascript
// Source object key: scary%name
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/scary%name' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👎 Fails with `Invalid copy source encoding` error

#### Plus literal:

```javascript
// Source object key: this+that
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/this+that' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👎 Fails with `key does not exist` error (because S3 is looking for `this that`)

#### Percent-encoded plus:

```javascript
// Source object key: this+that
(new (require('aws-sdk')).S3({ apiVersion: '2006-03-01' })).copyObject({ Key: '🆕', Bucket: 'acme-assets',
  CopySource: 'acme-assets/this%2Bthat' }, (data, error) => { console.log(JSON.stringify(data||error)); });
```

👍

Okay, that’s enough of that. What’s all that tell us? It reinforce the notion that S3 will gladly decode _any_ percent-encoded characters in a key. It also seems like for this esoteric `/my-bucket/my-object-key` input format, the permissive decoding applies to the entire value, not just the object key.

There’s also a bunch of examples where the value we’re providing actually _doesn’t_ need to be URL encoded. Is the JavaScript SDK doing some amount of encoding? It’s not encoding everything, or the emoji would work. This is ultimately making an HTTP request similar to the ones to looked at earlier, so there’s no way an actual space is making it out over the wire. Maybe it’s node?

It’s hard to say, and that’s kind of the point. Once you get behind enough layers of code, the predictability of how explicit you need to be in handling object key names goes out the window. Even once you pick a level of encoding that you want to implement, you’re likely relying more external code to handle the conversion.

Sticking with a JavaScript environment, consider the `this+that` example. If you use `encodeURI('acme-assets/this+that')` the result is `'acme-assets/this+that'`, which will not work properly. You could instead use `encodeURIComponent('acme-assets/this+that')` to get `'acme-assets%2Fthis%2Bthat'`, which proply encodes the `+`, but also encodes the `/`. That may not be a problem here, but in some cases, especially when presenting the value to a user, you may want to preserve the slashes for readability.

Or maybe you special-case the `+`, `encodeURI('acme-assets/this+that').replace(/\+/g, '%2B')`, to preserve the slashes in the path, even though `encodeURI` won’t encode `#`, which is not allowed in URL paths but we’ve seen make it through the SDK without issue. Neither standard JavaScript method for URL encoding does exactly what we want. You can imagine other languages, frameworks, and libraries having their own set of idiosyncrasies.

So what do we do with all this information?

One takeaway is that in any case where character limitations or handling could be a problem, encoding provides an escape hatch. We know there are places where S3 doesn’t decode inputs, like in the `Key` parameter of `copyObject` in JavaScript or `put_object` in Boto3, but those cases never have to worry about encoding. For the other cases, if you’re worried you can encode at will.

In those other cases, while you can encode everything, likely you won’t want to. Even if it’s code that’s deep in a library and just one computer talking to another, it’s really unintuitive to percent-encode an entire string, including alphanumeric characters. To stay on this side of crazy, consider preserving this set of characters: `A-Z`, `a-z`, `0-9`, `_`, `-`, `.`, and `/`. If you eliminate all the characters which are in any way considered unsafe by URL specs, HTML specs, S3 docs, etc, you’re left with that.

Percent-encode everything else, and you’ll generally still end up with reasonable, readable, safe strings to use when making requests to the S3 API, whether it’s through a browser, command line, SDK, etc. You’re fully protected from the weird `+` handling, though there will be some characters that get encoded, even though they almost certainly would work fine if not encoded. If you have specific needs where it’s important that particular characters remain unencoded, you probably need to manage that manually. For instance, if the pretty-safe-but-not-100%-safe `:` character has a special meaning in your app and should always be visible in object key strings or URLs, add it to the list of preserved characters in your encoder. If you try the same thing with the generally-not-safe-but-worked-with-copyObject `#` character, here be monsters.

If you consider the specific needs of your app, pay attention to the documentation of the AWS SDKs and APIs you’re dealing with (in particular, each parameter that deals with object key names), and understand the specifics of the encoding functions you’re using (or writing), you should be able to handle any object name without _too_ much trouble. It’s not uncommon to find people conforming or sanitzing file names on the way into S3, but hopefully this post provides enough guidance to make handling the full UTF-8 character set less work than the sanitzing would be.
