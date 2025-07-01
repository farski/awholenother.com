---
layout: post
title: IAM Conditions brain dump
description: Everything I know about AWS IAM conditions
date: 2025-02-17 09:26 -0500
reading_time: 40 minutes
tags:
  - AWS
  - IAM
---

While AWS does have pretty thorough documentation on [IAM policy variables](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html), and the various [context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceip) that can be used as policy variables, after reading through those guides several times something wasn’t clicking for me. These concepts are central to [ABAC](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html) on AWS. I could get things working, but not with full confidence or in a way that I could [explain](https://skeptics.stackexchange.com/questions/8742/did-einstein-say-if-you-cant-explain-it-simply-you-dont-understand-it-well-en).

Here are a bunch of example IAM policies that I reference to remind myself how some of the less obvious parts of this works in the wild, and some information about that whats and whys of how they work. (I‘m using YAML in these examples, because it‘s a bit more readable and compact than JSON, even though IAM policies are always JSON.)

This post assumes you mostly understand [IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html), and won’t get into the weeds on that.

---

This policy uses a wildcard in the resource. It allows creating buckets named like `acme-assets-images`, `acme-assets-pdfs`, etc. It doesn’t use any variables or conditions, but still offers some flexibility and dynamic behavior.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: acme-assets-*
```

---

This policy also uses a wildcard in the resource, but adds a [policy variable](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html).

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: arn:aws:s3:::${aws:PrincipalAccount}-*
```

The variable is one of the [global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceip), wrapped in `${…}`. (See below for information about service-specific context keys.)

Some context keys are considered **single-valued**, meaning when they are resolved they resolve to a single value. `aws:SourceIp` is an example of a single-valued context key; anywhere it is used, it will be substituted with the single IP address value of the current request. You can think of single-valued context keys as having a type of String, Number, etc if we were in a programming language.

Other context keys are **multivalued** context keys, meaning they (may) represent multiple values when they are resolved. `aws:TagKeys` is an example of a multivalued context key. We’ll look at `aws:TagKeys` more closely below, so the specifics don’t matter at this point, it’s just an example of a multivalued context key. You can think of these as having a type of Array in code.

When a context key is plural or is called something like `…List`, that’s generally a sign that it is multivalued. But you should [check](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) the key you are interested in if you’re not sure.

**Only single-valued context keys are allowed to be used as policy variables in a policy’s resource.** Multivalued context keys cannot be used as policy variables in a policy’s resource.

The policy above allows for creating buckets named like `123456789012-images` or `123456789012-db-backups`, where `123456789012` is the AWS account ID of the principal performing the create bucket operation (that’s what `${aws:PrincipalAccount}` resolves to). So, a principal belonging to account `111122223333` could create `111122223333-images` and a principal belonging to `999988887777` could create `999988887777-db-backups` with this same policy, since the `aws:PrincipalAccount` part of the allowed resource name is dynamic.

Other single-valued context keys work the same way. `Resource: arn:aws:s3:::${aws:SourceIp}-*` would allow a request coming from IP address `12.34.56.78` to create a bucket called `12.34.56.78-images`.

Interestingly, and perhaps not obviously, when using [_properties of the resource_](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resource-properties) as a variable, it seems that in most (all?) cases, those properties are available, even when the resource doesn’t exist, such as when creating a bucket.

For example, this policy allows creating a bucket in account `111122223333` called `111122223333-images`, even though that bucket doesn’t exist. You may think `ResourceAccount` would be missing or null until the resource exists, but that doesn’t seem to be the case. For create actions, the hypothetical future resource’s properties are included as part of the IAM request context, so you can use them.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: arn:aws:s3:::${aws:ResourceAccount}-*
```

---

One slightly more complex variation of the previous examples is when using the `aws:PrincipalTag/tag-key` or `aws:ResourceTag/tag-key` condition keys in your variables.

For these, the keys themselves change, based on which tag you are trying to reference. They take a form like `aws:PrincipalTag/environment` or `aws:ResourceTag/cost-center`, with the _key_ part of a tag coming after the slash.

(A reminder that AWS resource tags consist of two parts: a key, and a value.)

So for example, if you had an IAM role with the following resource tags:

- `environment`: `prod`
- `cost-center`: `security`
- `created-by`: `Alice`
- `aws:cloudformation:stack-name`: `prod-roles-stack`

These would translate to context keys like `aws:PrincipalTag/environment` or `aws:PrincipalTag/cost-center`, which could be used as policy variables like `${aws:PrincipalTag/environment}` or `${aws:PrincipalTag/cost-center}`.

A policy using tag-based context key variables in the resource would look like this:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: arn:aws:s3:::acme-${aws:PrincipalTag/environment}-*
```

This policy would allow the example role to create buckets like `acme-prod-images` or `acme-prod-db-backups`. Because the policy is dynamic, the same policy would allow a different role tagged with `environment=stag` to create `acme-stag-images`.

---

That’s about as complicated as things get when using IAM variables in policy resources.

It’s worth noting that you can use more than one variable in a resource:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: arn:aws:s3:::${aws:PrincipalAccount}-${aws:PrincipalTag/environment}-*
```

---

Things get more interesting, more useful, and more confusing once you start using policy conditions, especially conditions _with_ variables.

There’s a few things that contribute to the confusion.

The syntax can feel a bit awkward at times, because the [operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html) doesn’t come between the things you’re comparing.

For example, whereas you may be used to wrting an equality check like this in code:

```ruby
if (principalAccount == "111122223333")
```

The equivalent IAM policy would look more like:

```yaml
Condition:
  StringEquals:
    aws:PrincipalAccount: "111122223333"
```

You have to remember to mentally move the operator in between the JSON property and value, like `aws:PrincipalTag/environment StringEquals "prod"`.

For me, with equality operators like `StringEquals`, `NumericEquals`, I don’t have to work too hard to make sense of the condition, because we’re looking for the two values sitting next to each other to be the same. When using _inequality_ operators, like `StringNotEquals` or `NumericNotEquals`, my brain starts to wrinkle a bit. This is when mentally re-ordering things really helps. `"aws:PrincipalAccount" StringNotEquals "111122223333"` feels a lot more like `"aws:PrincipalAccount" != "111122223333"` to me than the actual condition format in the policy.

As you use IAM policy conditions more and more, this syntax becomes more familiar and requires fewer mental gymnastics to decipher.

Another point of confusion is once you start having multiples of things. Multiple policy statements with conditions, multiple conditions within a single statement, using some operator multiple times, multivalued context keys, or multiple values for single-valued context keys…

Both how to structure these cases of multiples, and how they interact (logical AND? logical OR?) can be tricky. The following examples will hopefully clarify all these different situations.

---

Let’s start by looking at a pretty simple policy:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:ResourceTag/environment: stag
```

This policy’s resource includes a wildcard, but no variables (just to reduce complexity; everything we covered about about variables in resources would still apply). We have now added a **condition block** to the policy statement. Each IAM policy statement can have a single condition block, which can contain one or more conditions. In this case, we have a single condition in this condition block.

Every condition tests some property of the request context against some value (or values) using an operator.

So in this case, we are comparing the value of the `environment` tag value of the target SQS queue to the static string value `stag`, using the `StringEquals` operator.

Hopefully that syntax structure is starting to make a little bit of sense, where the operator (e.g., `StringEquals`) precedes the two elements being tested.

This policy would allow principals to send messages to SQS queues with names starting with `acme-` when that queue is tagged with `environment=stag`.

---

It’s important to realize that in a condition like `{ StringEquals: { aws:PrincipalTag/environment: stag } }`, the `aws:PrincipalTag/environment` part is **not** a IAM policy variable. In an IAM condition, the left-hand side is _always_ a context key, meaning it’s always representing some dynamic value(s) from the request context. You **do not** make the left-hand side of the comparison an IAM policy variable.

```yaml
{ StringEquals: { aws:PrincipalTag/environment: stag } } # Correct
{ StringEquals: { ${aws:PrincipalTag/environment}: stag } } # Incorrect
```

---

We can expand the previous example a bit to see how we could introduce more tests to this policy statement (getting into some of those multiples I mentioned).

Let’s say we wanted to compare two different strings as part of this policy. Within a condition block, each operator (`StringEquals`, `NumericLessThan`, `DateGreaterThanEquals`, etc) can only appear once. But we can list multiple context keys under each operator to reuse that operator. So we could augment the previous example like this:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:ResourceTag/environment: stag
        aws:PrincipalTag/team: security # <- This was added
```

This policy is now performing two tests, both which are making string equality comparisons.

When a condition operator contains multiple context keys, those tests are [evaluated](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-logic-multiple-context-keys-or-values.html#reference_policies_multiple-conditions-eval) using logical `AND`.

So in this case, the policy is saying: allow sending SQS messages when the queue is tagged with `environment=stag` **AND** the principal is tagged with `team=security`. Both of those conditions must be met for this policy to allow the message to be sent.

---

Let’s say, instead, that you want to test only the principal’s `team` tag, but you want allow multiple teams to send these SQS messages. That would look something like this:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:PrincipalTag/team:
          - security
          - devops
```

This is an example of a condition testing a single-valued context key against a set of values. In this case, the policy would check that the principal is tagged with either `team=security` **OR** `team=devops`.

When a condition has a single-valued context key on the left-hand side, and multiple values on the right-hand side, the evaluation of that condition is done using logial `OR`.

Or to put it another way, the policy is checking if _any_ of the values in the set are a match with the context key’s value.

---

To continue to build a complex example, consider:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:ResourceTag/environment: stag
        # AND
        aws:PrincipalTag/team:
          - security
          # OR
          - devops
```

This policy allows sending SQS messages when the queue is tagged `environment=stag` **AND** [the principal is tagged either `team=security` **OR** `team=devops`].

---

Using multiple condition operators in a policy results in more `AND` evaluations.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:ResourceTag/environment: stag
      # AND
      DateGreaterThanEquals:
        aws:CurrentTime: "2020-01-01T00:00:00Z"
      # AND
      IpAddress:
        aws:SourceIp: 10.20.0.0/16
```

This checks that the queue’s `environment` tag equals `stag` **AND** the current time is after 2019 **AND** the source IP is in that `/16` block of IPs.

If we were to add more conditions to any of those operators, or were to provide a list of values for any of the conditions, they would work the same as in the more simple examples above (`AND` for multiple conditons, `OR` for a list of values).

---

Some operators are considered _negated matching_ operators, like `StringNotEquals`, `ArnNotLike`, etc. When using these operators with a list of multiple values, the evaluation of those values is made with logical `NOR`, rather than logical `OR`.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringNotEquals:
        aws:PrincipalTag/team:
          - hackers
          # NOR
          - interns
```

This policy will allow a principal to send a message to a queue so long as that principal doesn’t belong to either of the teams listed. The condition is satisfied when the `team` tag is neither `hackers` **NOR** `interns`.

Or to put it another way, the policy is checking if _none_ of the values in the set are a match with the context key’s value.

---

All the conditions we’ve written so far have used single-valued context keys. Even in cases where we have tested against a set of values, the context key itself represented a single value (like `aws:SourceIp` resolving to the single IP of the request, as mentioned earlier).

Let’s now create a condition that uses a multivalued context key.

`aws:TagKeys` is probably the most common multivalued context key you will see in examples, and that you will end up writing into your policies. We will look at `aws:TagKeys` shortly, but it has some quirks that make it a bad introduction to multivalued context keys. In fact, most of the multivalued context keys are pretty quirky, so to get started I’m going to invent something.

Imagine that our request context for some request looks like this:

- `aws:SourceIp`: `"12.34.56.78"`
- `aws:PrincipalAccount`: `"111122223333"`
- `aws:username`: `"alice"`
- `fake:Weather`: `["Sunny", "Warm", "Windy"]`

The `fake:Weather` context key is our made up example, and you can see that unlike the other elements of the context, it represents a set of values.

We want to write policy conditions that can evaluate these weather conditions in various ways. To do that we add _qualifiers_ to our condition operators, which provide the existing operators with the ability to operate on sets of data in the left-hand side of the comparison (the request context side). There are two such qualifiers: `ForAllValues` and `ForAnyValue`.

It’s very important to know that you **should never use these qualifiers with single-valued context keys**. IAM will allow you to write policies that do that, and it’s common to think you need them when testing sets of values against a singled-valued context key. It’s not necessary, and there are significant potential security issues that can be introduced. Only use set qualifiers with multivalued context keys.

Qualifiers are added to the front of operators. So we can take an operator like `StringEquals`, and turn it into a set-aware operator like `ForAllValues:StringEquals` or `ForAnyValue:StringEquals`.

Here’s an example of a policy that uses `ForAnyValue:StringEquals`:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAnyValue:StringEquals:
        fake:Weather:
          - Rainy
          - Humid
          - Warm
          - Cloudy
```

`ForAnyValue` works by looking for any value in the set of values in the context for the given context key (in this case, Sunny, Warm, and Windy) that satisfies the operator for any value in the set of values we provide in the condition (in this case, Rainy, Humid, Warm, and Cloudy). Because we are using the `StringEquals` operator in this example, which is looking for exact matches, and since we can find a match in the two sets (Warm), the condition is satisfied.

In other words, this policy will allow the request to send an SQS message when the weather for the request is at least one of Rainy, Humid, Warm, or Cloudy. Because the weather for our example context was Warm, the policy allows the message to be sent.

We can also include wildcards in the set of values we put in our policy, but we have to use the `StringLike` operator:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAnyValue:StringLike:
        fake:Weather:
          - W*
```

Here the policy is checking that at least one of the weather values in the context (Sunny, Warm, Windy) starts with a `W`. Since there is such a value (two, actually: Warm and Windy), this condition is satisfied for our example request.

Note how the set of values listed in the condition can be a single value.

Hopefully you can see how this pattern would apply to other operators when qualified with `ForAnyValue`.

The `ForAllValues` qualifier works in a similar way, but checks that _every_ value in the set of values from the context (Sunny, Warm, and Windy) satisfies the operator for some value in the policy condition’s set of values.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAllValues:StringEquals:
        fake:Weather:
          - Sunny
          - Cloudy
          - Warm
          - Cold
          - Windy
          - Calm
```

This policy would allow the request from our example context, because each of the weather conditions in the context appears in the policy. If another request was made that included `fake:Weather`: `["Sunny", "Warm", "Windy", "Humid"]`, this policy would not allow the request, since `Humid` would not match any of the values in the policy.

It’s pretty easy to think that `ForAllValues` also means that all the values in the policy condition set must have a match. That’s not the case. The `All` in `ForAllValues` is referring _only_ to the set of values in the context (the left-hand side).

Like with `ForAnyValue`, the policy could include a set that only contains a single value. Just realize that this a very restrictive policy. For example, `ForAllValues:StringEquals: { fake:Weather: ["Sunny"] }` would require that every weather condition present in the request is `Sunny`, which basically means that the _only_ weather condition present in the request is `Sunny`.

Hopefully you can see how this pattern would apply to other operators when paired with the `ForAllValues` qualifier, including using `StringLike` with wildcards.

---

There’s a major caveat to be aware of when usnig the `ForAllValues` qualifier: if the request context doesn’t include the context key your policy is testing, or if that context key is present but resolves to a null value or an empty string, the condition will always be satisfied, no matter what set of values you include in the policy.

For example, if we again consider this policy:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAllValues:StringEquals:
        fake:Weather:
          - Sunny
          - Cloudy
          - Warm
          - Cold
          - Windy
          - Calm
```

If that policy is evaluted for something like this context:

- `aws:SourceIp`: `"12.34.56.78"`
- `aws:PrincipalAccount`: `"111122223333"`
- `aws:username`: `"alice"`

Or this context:

- `aws:SourceIp`: `"12.34.56.78"`
- `aws:PrincipalAccount`: `"111122223333"`
- `aws:username`: `"alice"`
- `fake:Weather`: `""`

Then the policy will allow the SQS message to be sent, even though no weather conditions were actually matched.

In other words, the presence of a `ForAllValues` condition does not guarantee that the context key for that condition will actually resolve to any useful data.

When this behavior would be problematic, you should pair `ForAllValues` with the [`Null` condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null):

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      Null:
        fake:Weather: false # This condition is satisfied when fake:Weather IS NOT NULL
      ForAllValues:StringEquals:
        fake:Weather:
          - Sunny
          - Cloudy
          - Warm
          - Cold
          - Windy
          - Calm
```

Adding the `Null` operator allows you to ensure that the given context key doesn’t resolve to a null value, so that the overall policy behaves the way you’d expect, even when `ForAllValues` itself does not.

I find the `Null` operator to be the hardest of all operators to grok. The `Null: { fake:Weather: false }` condition is satisfied when `fake:Weather` is _not null_ – when `fake:Weather`’s null-ness is false. I just have to memorize it, I don’t think it will ever feel correct to me.

When using the `Null` operator with `ForAllValues`, you will generally only ever use `false`. There are other use cases of the `Null` operator where you’d use a `true` value, such as if you wanted a policy to only allow some action if a certain key doesn’t exist in the context (say, checking that `aws:SourceIdentity` isn’t present, if you don’t want any assumed roles to perform the action.)

You don’t need to use the `Null` operator with `ForAnyValue` in this same way. You may still have policies that use both, but it won’t be for dealing with this particular caveat.

---

Using `ForAllValues` and `ForAnyValue` can feel confusing when working with negated operators, so here are a some examples.

We’ll keep using this example context:

- `aws:SourceIp`: `"12.34.56.78"`
- `aws:PrincipalAccount`: `"111122223333"`
- `aws:username`: `"alice"`
- `fake:Weather`: `["Sunny", "Warm", "Windy"]`

Here’s a policy using `ForAnyValue:StringNotEquals`:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAnyValue:StringNotEquals:
        fake:Weather:
          - Rainy
```

This condition will be satisfied if there is any weather data in the request that does not equal `Rainy`. That’s the case here, since we can find a value in the request context that does not equal `Rainy` (in fact, we can find three). Additionally, we would still be able to find at least one value in the request context that doesn’t equal `Rainy` even if `Rainy` did appear in the request (like `["Sunny", "Warm", "Windy", "Rainy]`).

A slightly more complex example would include multiple values in the policy:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      ForAnyValue:StringNotEquals:
        fake:Weather:
          - Rainy
          - Snowy
```

The evaluation logic, though, is the same. The condition will be satisfied when any weather data in the request equals neither `Rainy` nor `Snowy`. For our example, we can find at least one value in Sunny/Warm/Window that is neither Rainy nor Snowy, so the condition is satisfied. Again, even if the request did include `Rainy`, or `Snowy`, or both `Rainy` and `Snowy`, the condition would be satisfied.

The only time where a `ForAnyValue:StringNotEquals` condition isn’t satisfied is when the context only includes values from the policy condition set. So for a request with `["Rainy"]`, or `["Snowy"]`, or `["Rainy", "Snowy"]`, or `["Rainy", "Rainy"]`, or `[]`, in these cases we can’t find any weather data from the request that don’t match, so the condition is not satisfied.

In other words, `ForAnyValue:StringNotEquals` checks for the presence of at least one value not in the policy. The presence of values in the policy don’t matter.

Now let’s look at `ForAllValues:StringNotEquals`, and start simple where the policy only has a single value in the condition:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      Null:
        fake:Weather: false
      ForAllValues:StringNotEquals:
        fake:Weather:
          - Sunny
```

This policy requires that `fake:Weather` is not null, and requires that no values from the request context match the value in the policy. For our example context (`["Sunny", "Warm", "Windy"]`), `Sunny` does appear, so the context does not satisfy this condition. As long as `Sunny` does not appear, this condition is satisfied. So `[]`, `["Rainy"]`, `["Warm", "Windy", "Partly Sunny"]` all satisfy the condition.

If we add more values to the condition:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      Null:
        fake:Weather: false
      ForAllValues:StringNotEquals:
        fake:Weather:
          - Sunny
          - Partly Sunny
```

The policy requires that none of the values listed appear in the request context. So our `["Sunny", "Warm", "Windy"]` context does not satisfy this condition. Neither does `["Partly Sunny", "Warm", "Windy"]`. `["Warm", "Windy"]`, `["Rainy", "Cloudy"]`, or any other context that excludes both `Sunny` and `Partly Sunny` would satisfy the condition.

In other words, `ForAllValues:StringNotEquals` check that none of the values in the policy are present in the request context.

---

So just to look at things all in one place:

- `ForAnyValue:StringEquals`: Requires that at least one value from the policy is present in the request. Does not limit any values from appearing in the request. Never requires that multiple values are present in the request.
- `ForAnyValue:StringNotEquals`: Requires that the request includes at least one value that isn’t present in the policy. Does not limit any values from appearing in the request.
- `ForAllValues:StringEquals`: Limits the values that can appear in the request to only those listed in the policy. Does not require that all values listed in the policy are present in the request.
- `ForAllValues:StringNotEquals`: Prohibits the request from containing any values listed in the policy. Does not require any values to appear in the request.

---

Okay. With all that sorted out, we should have a solid foundation. There’s one big puzzle piece that hasn’t been covered yet, and that’s using policy variables within conditions.

From a conceptual standpoint, this may not seem like a big thing, and it isn’t. We will use variables in conditions the same way we used them in resources at the beginning of the post. And conditions will behave the same with variables as they did with static values.

The reason that variables-in-conditions carries a lot of weight is because it is the cornerstone of ABAC in AWS.

As we’ve seen, conditions allows us to write policies based on aspects of the current request context. Using policy variables in conditions allows us to compare one aspect of the context to another aspect of the same request context. This is really powerful, since it means that both sides of the equation are now dynamic.

---

Let’s start with something pretty basic:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:SendMessage
    Resource: acme-*
    Condition:
      StringEquals:
        aws:PrincipalOrgID: ${aws:ResourceOrgID}
```

This policy compares the organization ID of the principal performing an SQS `SendMessage` operation (i.e., the organization the principal belongs to) with the organization ID of the resource the message is being sent to (i.e., the organization the SQS queue belongs to). When those values are equal (i.e., the principal and the queue belong to the same organization), the action is allowed.

Notice how the left-hand side (`aws:PrincipalOrgID`) is a bare context key, and the right-hand side is a policy variable (`${aws:ResourceOrgID}`).

All of the previous examples we’ve looked at for building more and more complex conditions apply when you start to mix in variables. You do have to be careful, though, because not all operators allow variables to be used. It’s pretty rare that you’ll try to write a condition using a variable where it’s not supported, but it’s not impossible. You can look at the operator reference to see which do and do not.

---

As a quick recap:

- In a policy’s resource, string literals and IAM policy variables can be used interchangeably, depending on your needs.
- In a policy’s condition, the left-hand side is always a context condition key (never a literal value or a IAM policy variable).
- In a policy condition, the right-hand side can be either a literal value(s) (string, number, list, etc) or an IAM policy variable, depending on your needs.

---

When we talk about implementing ABAC in AWS, generally what we mean is writing policies that use conditions and variables to compare resource tags on the principal and resource in a request context. In AWS, tags _are_ the attributes in ABAC.

For example, let’s say you want a policy that allows principals with the `team=security` tag to be able to create any S3 buckets they want:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:CreateBucket
    Resource: "*"
    Condition:
      StringEquals:
        aws:PrincipalTag/team: security # Check if the principal making the request is tagged with `team=security`
```

If you also want to allow buckets to be deleted, but you want to ensure buckets used in production are protected from being deleted:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:DeleteBucket
    Resource: "*"
    Condition:
      StringNotEquals: # Note the "Not" in StringNotEquals
        aws:ResourceTag/environment: prod # Check that the bucket being targeted does **not** have an `environment` tag equal to `prod`
```

These two examples highlight the differences in using attributes of the principal making the request and attributes of the resources the request is targeting.

The two methods will often be combined. For example, if you wanted a policy that allows a team to delete any of their own buckets:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: s3:DeleteBucket
    Resource: "*"
    Condition:
      StringEquals:
        aws:ResourceTag/team: ${aws:PrincipalTag/team} # Compare the `team` tag on the requesting principal to the `team` tag on the target resource, and allow the deletion if they match
```

If this is all feeling very familiar, because of how many examples we’ve already looked at, that’s good, and it should.

---

Quick side note: For requests where both the identity policy and resource policy are being used to determine access (see [evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)), remember that both policies are evaluated in the same context. So while there may be context keys that seem more relevant to one or the other – they aren’t. The `aws:Principal*` properties work in both identity and resource policies, and, for any given request, the value of any given context key will be the same in both policies. Same thing goes for `aws:Resource*` properties.

So if you find yourself thinking that you need, for example, to use `aws:ResourceAccount` in a resource policy, and `aws:PrincipalAccount` in an identiy policy, you may want to revisit how the [request context](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-request) works, and the evaluation logic.

---

In addition to the `aws:PrincipalTag/` and `aws:ResourceTag/` context keys, there’s also `aws:RequestTag/`. Whereas `PrincipalTag` and `ResourceTag` are used to refer to resource tags that have already been applied to principals and resources, and are helpful in deciding who can do what, or which things they can operate on, `aws:RequestTag/` is used to control the actual process of tagging resources.

Just like the others, it takes a form like `aws:RequestTag/team` or `aws:RequestTag/cost-center`.

`aws:RequestTag/` is relevant for operations where tags are being changed.

There are some operations where `aws:RequestTag/` is not relevant. For example SQS’s `SendMessage` does not create, update, or delete resource tags. It cannot; that is not a function of that particular API call. For operations like this, `aws:RequestTag/` does not provide any real benefit.

It’s important to remember that `aws:PrincipalTag/` and `aws:ResourceTag/` are hugely useful for these types of operations, though. As we’ve seen in many examples above, we can control how SQS messages get sent using principal and resource tags in our policies.

`aws:RequestTag/` gives you access to the tags that are being created, updated, or deleted as part of tagging operations. So while SQS’s `SendMessage` does not do any tagging, `TagQueue` and `UntagQueue` do. Many AWS services have these sort of tagging APIs: SNS’s `TagResource` and `UntagResource`, EC2’s `CreateTags` and `DeleteTags`, etc.

Some services support tagging but don’t have specific API endpoints for managing the tags. CodeBuild, for example, does not have any sort of explict tag or untag APIs. You manage tags as part of other operations. When you use `CreateProject` or `UpdateProject`, you manage the project’s tags as part of those operations.

Whether a service has explicit tagging operations, or they are encompassed within other operations, or a combination of the two, those operations are the ones where `aws:RequestTag/` comes into play. The request context for those operation requests will include the context keys for the tags that are being created or updated.

You can look at the Service Authorization Reference document for a service to see which actions will include request tags in the context.

---

There’s a couple different reasons why it’s important to control how resources are tagged in AWS.

One is consistency. You may have a tagging system and want to ensure that resources don’t deviate from that. Here’s an example of how you may create a policy that only allows certain values to be used for the `environment` tag on SQS queues and SNS topics.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action:
      - sqs:TagQueue
      - sns:TagResource
    Resource: "*"
    Condition:
      StringEquals:
        aws:RequestTag/environment:
          - test
          - stag
          - prod
```

Note, however, that this policy does **not** control who can do the tagging, or which resources they can tag. This policy only controls what that one particular tag can be. It would allow, for example, someone to change the `environment` tag of some resource from `test` to `prod`. You may not want everyone to be able to do that, so you may also add other conditions to restrict tagging, beyond just which values can be used.

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action:
      - sqs:TagQueue
      - sns:TagResource
    Resource: "*"
    Condition:
      StringEquals:
        aws:RequestTag/environment:
          - test
          - stag
          - prod
        aws:PrincipalTag/team:
          - devops
```

Now our policy only allows members of the DevOps team to make those tagging changes, and even they are still restricted as to which values they can use.

The second, and by far the more important, reason is that once you start using tags as the basis for your access control, controlling the tags themselves becomes critical. This goes beyond consitency for the sake of consistency or something like billing. It determines who has access to private data, who can turn servers on and off, who can make decisions that cost many thousands of dollars.

---

`aws:RequestTag/` is one part of that system for controlling tags. A separate context key called `aws:TagKeys` is the other.

`aws:TagKeys` is relevant to tagging operation requests, just like `aws:RequestTag/`. Unlike `aws:RequestTag/`, which is a single-valued context key where you reference each tag separately (e.g., `aws:RequestTag/team` and `aws:RequestTag/environment`), `aws:TagKeys` is a multivalued context key.

In the request context for any tagging operation, `aws:TagKeys` will be a list of _just the keys_ of any tags that were included in the operation. This does **not** include the principal or resource tags that exist on the principal making the request, or the resource being targetted.

So imagine there’s an IAM role making a request, and that role has been tagged `team=security`. The role is making an SQS `TagQueue` request to add a `cost-center=manufacturing` tag to an existing queue. That queue has already been given a resource tag of `environment=prod`.

In this case, the request context would include the `aws:TagKeys` context key, which would have a value of `["cost-center"]`. The `team` and `environment` tag on the principal and resource are not included in `aws:TagKeys` (though they would be available using `aws:PrincipalTag/team` and `aws:ResourceTag/environment` as always).

The entire context may look like this:

- `aws:SourceIp`: `"12.34.56.78"`
- `aws:PrincipalAccount`: `"111122223333"`
- `aws:PrincipalTag/team`: `"security"`
- `aws:ResourceTag/environment`: `"prod"`
- `aws:RequestTag/cost-center`: `manufacturing`
- `aws:TagKeys`: `["cost-center"]`

As a multivalued context key, we can use the `ForAnyValue` and `ForAllValues` operation qualifiers to comprehensively control which tags can be added to resources with `aws:TagKeys`.

All the various ways that those qualifiers allow us to require values, restrict values, prevent certain values, etc, we can use to control which tags are added to resources.

---

It’s important to remember that `aws:TagKeys` and `aws:RequestKey/` come into play only during tagging operations. They do not necessarily ensure that resources are tagged or tagged in any particular way, even if those resources are being manipulated in other ways.

For example, imagine you have an existing SQS queue that doesn’t have any tags, and you create a policy that requires queues to be tagged with a `team` and `environment` tag, perhaps with some limited set of values for each, for the `sqs:TagQueue` and `sqs:UntagQueue` actions. If a user comes along and calls `SetQueueAttributes`, that action is not governed by the tagging policy (nor could it be, since it’s not a tagging operation), so the attributes will be updated but queue will continue to be untagged.

---

Similar to how we can add `ForAnyValue:` and `ForAllValues:` to condition operators to create set-aware operators, we also have the option of adding `…IfExists` to operators, which makes the condition apply only if the given context key is present in the request.

Take this example:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: ec2:*
    Resource: "*"
    Condition:
      StringEqualsIfExists:
        ec2:InstanceType:
          - t4g.small
          - t4g.medium
```

This policy is ensuring that only two particular types of EC2 instances can be used. If it had used the standard `StringEquals` condition, most EC2 operations would be blocked by this policy, since most operations don’t include the `ec2:InstanceType` context key. For example, if someone tried to call `ec2:CreateSubnet`, that request wouldn’t include any `InstanceType` information, since it’s not an instance-based operation. We aren’t actually trying to prevent that request, and by using `StringEqualsIfExists` we can have this condition ignored, until it’s being used for an instance-based operation.

By creating the policy this way, we aren’t responsible for curating a list of all instance-based operations in the policy’s `Action`. If, over time, new actions are added that are considered instance-based, and IAM includes the `ec2:InstanceType` context key in those requests, the policy will kick in and continue to control which instance types are allowed.

Note that `Null` does not have an `…IfExists` variant.

---

The previous example illustrates a service specific context key (`ec2:InstanceType`). The other examples we’ve looked at so far used global context keys, which are available in all requests contexts, regardless of which AWS service is handling the operation.

Many services, though, can add context keys that are only relevant within that service, such as EC2’s intance type. You can look at the service’s service authorization documentation (e.g., [EC2](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2.html#amazonec2-policy-keys)) to see if there are any service-specific keys.

You will probably notice that many of these documents will list some (but not all) global keys. For instance, [EC2](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2.html#amazonec2-policy-keys) lists `aws:TagKeys` explicitly, even though it’s a global condition key. I don’t have a good explanation for why they do that.

You’ll also come across some duplication between global and service-specific keys. For example `ec2:ResourceTag/${TagKey}` and `aws:ResourceTag/${TagKey}`. I believe these service-specific keys are maintained for backwards compatibility, and the values should be identical in cases where both are available. You may find reasons to reference the service-specific keys, though, even when a global key is available (like if you wanted to write a policy that applied to all services, but only dealt with tagging for EC2 resources).

---

You may be thinking that combining `ForAnyValue` and `…IfExists` would let you do some useful things.

Consider this policy:

```yaml
Version: "2012-10-17"
Statement:
  - Effect: Allow
    Action: sqs:*
    Resource: "*"
    Condition:
      ForAnyValue:StringEqualsIfExists:
        aws:TagKeys:
          - team
```

On paper, it would appear that this policy is ensuring that the `team` tag is present on tagging operation requests (those where the `aws:TagKeys` key is present), and wouldn’t affect other, non-tagging operations (like `sqs:SendMessage`).

Unfortunately, **this policy does not work that way**. IAM resolves the `ForAnyValue` part of the condition before `…IfExists`, and `ForAnyValue` conditions can’t be satisfied when the given context key is missing or null.

So in the case of an `sqs:SendMessage` operation, the request context will never have `aws:TagKeys`, which means `ForAnyValue:StringEqualsIfExists: aws:TagKeys` is _always_ false.

In other words, `ForAnyValue:StringEqualsIfExists` is treated the same as `ForAnyValue:StringEquals`, and you should avoid using `…IfExists` with other qualifiers to prevent confusion.

---

So now we have two tools to help control tagging. We have `aws:TagKeys`, which controls _which_ tags can be used, and we have `aws:RequestTag/` which controls which values can be used for those tags.

The building blocks of wildcards, variables, and conditions offer a lot of flexibility when it comes to writing powerful authorization policies in AWS, but there are gotchas and rough edges, so you do have to care when venturing into more complicated territory.

### Relevant AWS documents

- [Global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-resource-properties)
- [AND/OR logic for multiple conditions/operators/values](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-logic-multiple-context-keys-or-values.html#reference_policies_multiple-conditions-eval)
- [Condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_ARN)
- [Single-value and multivalued context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-single-vs-multi-valued-context-keys.html)
- [ForAll and ForAny](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-single-vs-multi-valued-context-keys.html#reference_policies_condition-multi-valued-context-keys)
- [IfExists](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_IfExists)
- [Null](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Null)
- [Policy variables](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-wheretouse)
- [Using tags for ABAC](https://docs.aws.amazon.com/tag-editor/latest/userguide/tags-in-iam-policies.html)
- [IAM service-specific context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_iam-condition-keys.html)
- [Controlling access to AWS resources using tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html)
