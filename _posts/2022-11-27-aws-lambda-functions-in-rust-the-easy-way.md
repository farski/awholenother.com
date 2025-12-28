---
layout: post
title: AWS Lambda functions in Rust the easy way
date: 2022-11-27 14:57 -0400
update: 2023-03-01 07:34 -0500
reading_time: 20 minutes
tags:
  - AWS
  - Lambda
  - Rust
---


**Update:** As of March 2023, the AWS SAM CLI now offers [native support](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/building-rust.html) for building Rust-based Lambda functions. The underlying mechanisms are largely the same as what is described in this post, but `BuildMethod: makefile` has been replaced with `BuildMethod: rust-cargolambda`. I haven’t had a chance to try this yet. Once I do, I will update the post to reflect this new functionality. The Rust code described in the post is still accurate.

<hr>

Deploying Rust code to an AWS Lambda function is a topic that has been written about before, but nothing approached it quite the way I was looking for. I have a lot of experience with AWS, Lambdas, and deploying things to AWS and Lambdas. I have pretty limited experience with Rust. But I’ve read a bunch of tutorials, and sort of understand the basics and can make it do some things. That’s the level of Rust exposure this post assumes you have.

Most of what I’m programming these days are web services: building APIs, or writing software that ties various APIs together. And nearly all of what I work on runs in the cloud. The best sandbox for me for learning Rust is a similar public cloud environment, where there are lots of different things to interact with (files, databases, queues, etc) readily available. For better or worse, I learn the fastest when I can learn it in the context of AWS.

To that end, I wanted to have something that allowed me to write and deploy Rust code as easily as I could write and deploy Node.js code. When I want to prototype something in Node, all I need is an `index.js` file with some code and a `template.yml` CloudFormation template, and running `sam build && sam deploy` gets the wheels turning. As a learning or sandbox environment, that gets me access to all aspects of the AWS cloud and the freedom to start playing around with whatever new service, API, or language feature has caught my attention. Extremely low friction.

The goal is to get as close to that workflow for Rust as possible, and it turns out we can get pretty close, even though the two ecosystems are quite different.

> There are many differnt ways to deploy code to Lambda functions or other serverless platforms, and I suspect many could provide a similar developer experience. AWS CloudFormation and AWS SAM are the systems I’m most familiar with, which is why I’m focusing on them.

## Node.js

Let’s quickly review what happens when deploying a very simple Node.js app with AWS SAM.

We’d start by creating a code file which, by convention, would be called `index.js`. This example doesn’t do anything interesting, but it would run when invoked and return a value.

```javascript
export const handler = async (event, context) => {
	return { event, context };
};
```

Next, we’d create a SAM template (generally named `template.yml`), which includes the defintion of the Lambda function we want to deploy into AWS.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: This template deploys a pretty useless serverless application

Resources:
  UselessFunction:
    Type: AWS::Serverless::Function
    Properties:
      Architectures: [arm64]
      CodeUri: ./index.js
      Description: >-
        A useless function for a useless app.
      Handler: index.handler
      MemorySize: 128
      Runtime: nodejs18.x
      Timeout: 8

  UselessFunctionLogGroup:
    Type: AWS::Logs::LogGroup
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      LogGroupName: !Sub /aws/lambda/${UselessFunction}
      RetentionInDays: 30
```

This template asks AWS to create a Lambda function with some basic settings (as well as a log group for the function, to follow best practices).

Since this application is using one of Lambda’s native runtimes (Node.js 18), really all that needs to happen to get it into the cloud is to get `index.js` and `template.yml` somewhere where they can be deployed. The AWS SAM utility does that for us. `sam build` takes this template as we’ve written it, with references to files relative to our local file sysmem (e.g., `CodeURI: ./index.js`), and then uploads those referenced files to S3, and then creates a new copy of the template where the local references have been replaced with references to those objects in S3.

Then, when you run `sam deploy`, that newly-built template can be sent to CloudFormation, which will create the Lambda function using the code file in S3, and the log group that we asked for.

> There is some initial configuration that I didn’t cover, which tells `sam` where you want to deploy things. Those steps are not really relevant to deploying a Rust app.

The application code could grow to multiple files, the template to grow to include many functions and other types of resources, but the process for deploying a Node.js app with any level of complexity using this system remains pretty much the same.

## Rust

There are two reasons why a Rust-based workflow can’t work the same way as with Node.js:

The first is that Rust code must be compiled. The Node.js runtime will interpret JavaScript code just as we’ve written it on the fly. We could even change the Lambda from the `arm64` architecture to its `x86_64` counterpart, and that JavaScript code we wrote would continue to work. Rust code is not very useful until it’s been compiled for the specific plaform that it’s going to run on. So that’s one additional step we’re going to have to add.

The other reason is that AWS Lambda doesn’t natively support Rust, even if it’s been compiled. If you simply create a Lambda function with a binary compiled from some Rust code, the service won’t know what to do with it. We need to create a Lambda function in a way where it knows how to talk to our binary, and our binary needs to be able to talk back. Again, this introduces some extra steps as compared to the Node.js example, but as you’ll see, not that much extra complexity. After some one-time setup, the developer experience ends up in a very similar place to Node.js or other native runtimes.

### The Binary

There are two files we’re going to need on the Rust side of things: `Cargo.toml` and `main.rs`.

If you have started learning about Rust at all, you’ll be familiar with `Cargo` files. They define settings related to how your code will build, as well as project dependencies. For getting Rust code running in a Lambda, there are a few important things that need to be present in the `Cargo.toml` file. Below is the complete TOML file, and below that the important aspects are explained in further detail.

```toml
[package]
name = "echo"
version = "0.1.0"
authors = ["Christopher Kalafarski <chris@farski.com>"]
edition = "2021"
autobins = false

[dependencies]
tokio = { version = "1", features = ["full"] }
lambda_runtime = "^0.7.0"
serde = "^1"
serde_json = "^1"
serde_derive = "^1"
tracing = "^0.1.36"
tracing-subscriber = { version = "^0.3.15", features = ["env-filter"] }

[[bin]]
name = "bootstrap"
path = "./main.rs"
```

At the time of writing, `edition = "2021"` was the most recent Rust edition that was compatible with the other things necessary to make this all work.

Setting `autobins = false` disables the automatic target discovery that happens by default when compiling binaries with Cargo. In order to create a binary that matches the Lambda service’s expectations for how binaries are named, the configuration for the output binary will be explicitly defined further down the Cargo file.

As far as I know, `tokio` is the best option currently for an async Rust runtime on Lambda. Others may also work, but I have not tried them.

The line of code in this file that does the most heavy lifting is `lambda_runtime = "^0.7.0"`. `lambda_runtime` is a Rust [project maintained by AWS](https://github.com/awslabs/aws-lambda-rust-runtime) that does pretty much all the work required for creating a binary that knows how to "talk back" to the underlying Lambda platform.

If you think about our original Node.js-based Lambda function, there are three layers: the Lambda service, the Node.js runtime that’s offered natively, and the JavaScript code that we provided. The Lambda service is responsible for things like provisioning resources needed in AWS data centers to run your code and ultimately controls data into and out of a function execution. The runtime knows how to interact with that service layer, and also knows how to interpret JavaScript code. The JavaScript code is our business logic that the runtime consumes.

Rust is a bit different, since there is no middle layer interpretting code on the fly. Instead, we need to teach out binaries to interact with the Lambda service layer directly, at the same time as they are taking care of our business logic. You could read up on the API for building custom Lambda runtimes and implement that interface yourself, but `lambda_runtime` provides a stable, nearly first-party alternative. It provides a few primatives that we can implement in our own Rust code to ensure that inputs coming from the service layer and outputs we’re sending back are handled correctly, and work in the same way that the native runtimes like Node.js and Python would.

To put it another way, our binary _is_ the custom runtime, it’s not something we’re running within a custom runtime.

```toml
serde = "^1"
serde_json = "^1"
serde_derive = "^1"
```

The `serde` crates are not strictly necessary for making a Rust-based Lambda function. They are a great option for dealing with event I/O during Lambda executions, which is so common that I think including them in this first example gets things much closer to a real-world application.

```toml
tracing = "^0.1.36"
tracing-subscriber = { version = "^0.3.15", features = ["env-filter"] }
```

Again, these crates are here more for illustrating something fundamental to writing a good Rust-based Lambda, and not strictly required for basic functionality. These `tracing` crates are used for logging, and work in a way that aligns with Lambda’s expectations, so they are a good fit. There are other logging libraries that would work.

```toml
[[bin]]
name = "bootstrap"
path = "./main.rs"
```

Since we disable automatic binary targets earlier (`autobins = false`), we use the `[[bin]]` directive to explicitly tell the compiler to create a binary called `bootstrap` (which is what AWS Lambda will expect, when using custom runtimes).

### The Code

With the Cargo file in hand, we now have a few basic tools to help us create a custom Lambda runtime using Rust. We’ll be able to serialize and deserialize values in a way that’s compatible with the platform, using `serde`, and we can do some logging very reminiscent of how you would use `console.log` or `console.debug` with Node.js.

Below is the complete `main.rs` file with inline comments explaining what’s going on. This is intended to be a basic example that primarily illustrates how to integrate the `lambda_runtime`, but as mentioned before, doesn’t completely gloss over other fundamental aspects, like logging, that you’d want in a more complete app.

```rust
use lambda_runtime::{service_fn, LambdaEvent, Error};
use serde::{Deserialize, Serialize};
use tracing::info;
use tracing_subscriber::EnvFilter;

// This EchoRequest struct illustrates an input event that the Lambda function
// would receive. In this case, the input has a single `message` property. In
// reality, you would have a struct that matched your app's actual payload, or
// you may use a data structure provided by a crate for common events coming
// from other AWS services, like API Gateway or SNS (see the aws_lambda_events
// crate).
//
// Deserialize is provided by serde, and takes care of deserializing the raw
// input (i.e., JSON) into a native Rust struct, making it easy to work with in
// our code.
#[derive(Deserialize)]
struct EchoRequest {
  message: String,
}

// Similar to EchoRequest, this is an illustrative data structure that
// represents a simple output for this Lambda function. In reality, this would
// be something that reflects the payload your code is ultimately delivering.
//
// Again, serde is being relied on to handle serializing the native Rust data
// into something that the Lambda service layer expects to receive from a
// runtime (i.e., JSON).
#[derive(Serialize)]
struct EchoResponse {
  echo: String,
}

// The standard main() function of the binary is responsible for bootstrapping
// the Lambda runtime. It returns a basic Rust Result, and as you can see is
// not expected to return any significant value. The return value of main() is
// *not* the return value of a Lambda function execution.
#[tokio::main]
async fn main() -> Result<(), Error> {
  // tracing_subscriber is not strictly required to get logging working with
  // Rust-based Lambda functions, but many crates that you may end up using,
  // including the AWS SDK, use the tracing framework. Using tracing_subscriber
  // will log activity from those libraries.
  tracing_subscriber::fmt()
    // Filters logging based on the environment default, which is based on the
    // value of the RUST_LOG environment variable (which, for our Lambda
    // function, is set in template.yml). You could also filter logs on a
    // module-by-module basis of libraries you're using.
    .with_env_filter(EnvFilter::from_default_env())
    // This needs to be set to false, otherwise ANSI color codes will
    // show up in a confusing manner in CloudWatch logs.
    .with_ansi(false)
    // Disabling time is handy because CloudWatch will add the ingestion time.
    .without_time()
    .init();

  // The run() function provided by lambda_runtime starts polling for events
  // from the Lambda service. It basically does this indefinitely, for the
  // lifetime of the runtime.
  // service_fn() is a function provided by lambda_runtime, and wraps a
  // function that we define in a format known to the runtime.
  // event_handler is an arbitrary function or closure where our own
  // application code lives, and where the event for any given function
  // execution will be handled.
  lambda_runtime::run(service_fn(event_handler)).await?;
  Ok(())
}

// This is the meat and potatoes of our Lambda function, similar to
// handler = async (event, context) => {}; from the Node.js example. We saw
// above that this function is indirectly being passed into the run() function
// of the runtime. Every time this Lambda function is invoked, this Rust
// function will be called with the event data for that invocation.
//
// This function can be named anything. The Lambda function is not configured
// to call any particular function in app code, like you would see with the
// Node.js or Python runtimes.
//
// The LambdaEvent is a type that generically represents the event and
// execution context. By giving it the EchoRequest type that we defined
// earlier, we're able to tell the handler that the event data has a more
// specific format than the generic LambdaEvent would provide.
//
// EchoResponse is given as the Ok() type of the Result return value, so the
// handler will always return a value that matches the struct we defined above.
async fn event_handler(event: LambdaEvent<EchoRequest>) -> Result<EchoResponse, Error> {
  // The info! macro provided by tracing makes it easy to generate
  // Lambda-friendly and CloudWatch-friendly logs from our app. This is similar
  // to console.log in a Node.js app, and you could use other log-level macros
  // like debug! or error! as needed.
  info!("Starting Lambda event handler for request {}", event.context.request_id);

  // Constructs a basic response using the struct we defined earlier. Since the
  // Serialize trait was added to the struct, it will automatically be
  // serialized into a JSON format that is safe to leave the runtime as the
  // output of the execution.
  let resp = EchoResponse {
    echo: format!("{}… {}… {}…", event.payload.message, event.payload.message, event.payload.message),
};

  Ok(resp)
}
```

### Deploying

We’re now less than 10 lines of code away from being able to `sam build && sam deploy` this app.

We can take the CloudFormation template from the original Node.js example, and make a few small changes to tailor it to our Rust example. Those changes are commented inline below.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: This template deploys a working Rust-based Lambda function

Resources:
  EchoFunction:
    Type: AWS::Serverless::Function
    Properties:
      Architectures: [arm64]
      # This assumes that main.js is located at src/echo/main.js, relative to
      # template.yml (or wherever you're going to run `sam build`).
      CodeUri: ./src/echo
      Description: >-
        A function running on a custom Rust runtime
      Environment:
        Variables:
          # Enables backtrace output when the app panics (for debugging)
          RUST_BACKTRACE: '1'
          # Sets the log level that will make it through the filter we
          # set using with_env_filter()
          RUST_LOG: info
      # This is the name of the binary executable that holds the runtime and
      # our application code. We used the [[bin]] directive in Cargo.toml to
      # ensure that the compiler creates a binary file named `bootstrap`.
      Handler: bootstrap
      MemorySize: 128
      # Unlike with native runtimes like Node.js or Python, where the value
      # of this property would select a specific runtime version, when set to
      # `provided` or `provided.al2`, it's essentially choosing **no**
      # runtime, and instead saying we'll be providing the runtime. The
      # `al2` indicates that we want that runtime to run within an Amazon
      # Linux 2 environment (as opposed to Amazon Linux version 1).
      Runtime: provided.al2
      Timeout: 3
    Metadata:
      # The AWS SAM command line tool will detect this metadata and invoke a
      # Makefile in the CodeUri directory during the build process. The result
      # will be a Zip file that gets deployed as the Lambda function's code.
      # We'll want our compiled binary to be in that Zip file.
      # The file will be pushed to S3, and a reference to it will replace the
      # local file reference in the template, just as we saw with the Node.js
      # example earlier.
      BuildMethod: makefile

  EchoFunctionLogGroup:
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub /aws/lambda/${EchoFunction}
      RetentionInDays: 30
```

The final piece of the puzzle is the `Makefile` that our build process is now relying on, per the `BuildMethod`.

> The paths in this example expect both `main.rs` and the `Makefile` to live in something like `src/echo`, and for `template.yml` to live at the root of the project. That’s probably a pretty standard directory structure for most Lambda apps, and would allow for more functions to be added (e.g., `src/auth`, `src/export`, etc), each with their own `Cargo.toml`, `main.rs`, `Makefile`, and other files specific to each individual Lambda function.

The `Makefile` is pretty simple: it compiles our Rust code to a binary using `cargo lambda`, and then copies the binary into the location that the AWS SAM CLI expects it to be at the end of the build process (which it provides as `ARTIFACTS_DIR` when it calls `make`).

[Cargo Lambda](https://www.cargo-lambda.info) is a project that handles all the messy bits related to cross compiling Rust code intended for Lambda functions. Regardless of which OS or platform you’re developing on, and which platform you’re targeting, `cargo lambda build` should _just work_. It uses [Zig](https://ziglang.org) rather than VMs or Docker, and in my experience works much more reliably than other options I’ve seen for creating Lambda-ready binaries on an M1 Mac.

Follow the instructions on their website to install Cargo Lambda (using Homebrew, if you’re on a Mac).

```makefile
# NOTE: Paths are relative to the SAM template, not this Makefile.

build-EchoFunction:
	cd ${PWD}/src/echo && cargo lambda build --arm64 --release
	cp ${PWD}/src/echo/target/lambda/bootstrap/bootstrap $(ARTIFACTS_DIR)
```

## Wrap Up & Next steps

Once you’ve configured your SAM deployment, your project should have five files:

- `template.yml`
- `samconfig.yml`
- `src/echo/Cargo.toml`
- `src/echo/main.rs`
- `src/echo/Makefile`

A simple `sam build && sam deploy` should get you a compiled, deployed Lambda function that works just like any other Lambda function. From there you can start expanding your serverless applications using the same principals you’d follow for any other Lambda-based app.

One of the most obviously differences between a Rust-based Lambda function and one using Node.js, is that you can’t edit the code in the AWS Console. So those times where you want to quickly add some logging or hotpatch a bug will require a full recomplile and redeploy. Depending on how long it takes your project to compile, this could be fairly time consuming, so you may find that you need to be a bit more deliberate about code changes. Especially in the context of my needs that spurred this post: experimentation. When you’re working with a new service or API and need to trial-and-error some configuration, with a Node.js Lambda function, you can make small code changes in just a couple seconds. With Rust, it’s not so easy.

Given that this is a much more low-level developer experience, there are countless places to look for efficiency gains, though. The Lambda runtime docs touch on using [feature flags in lambda_http](https://github.com/awslabs/aws-lambda-rust-runtime#feature-flags-in-lambda_http), which is just one way you can reduce compile times and iterate faster.

I have been following the landscape of Rust-based Lambda deployments for a while, and while it seems to have solidified quite a bit, its short history has seen some instability. The runtime API has changed fairly significantly a couple times between versions. The toolchain for building binaries has seen several different Docker-based approaches now replaced with Cargo Lambda. The various pieces now seemed stable and reliable enough to write about, but I would anticipate there will be breaking changes to parts of the system over time.

Because of that churn, lots of what has been written about in other blogs, on StackOverflow, etc on this topic is anywhere from slightly different to entirely outdated. Even today, there are a number of different ways to approach this problem using the most modern tools. The README for the Rust runtime itself talks about deployment options using Cargo Lambda and AWS SAM that aren’t the same as how I’ve talked about them here. I’ve tried nearly every version of this that I’ve come across, and what I described above seems to be the easiest, and the one that matches other Lambda deployments most closely. The Makefile that SAM uses nicely encapsulates the Rust-specific bits, and other than that is very similar to building a Node.js or Python Lambda app. I would make sure to check dates on things you read, so that you don’t start using already-obsolete code snippets in your brand new app.

Hopefully this process allows you to get started with running Rust in Lambda without much trouble, and hopefully that gives you more opportunities to learn and practice Rust, as it did for me. Many of the concepts discussed in this post could also be extended to less experimental settings, like a real CD pipeline.
