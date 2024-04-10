---
layout: post
title: AssumeRole vs. AssumeRole vs. PassRole
date: 2024-04-06 19:03 -0500
tags:
  - AWS
  - IAM
---

<small>Reading time: 20 minutes</small>

When working with AWS [IAM](https://aws.amazon.com/iam/), granting permissions is usually pretty straightforward: you want someone (a user, an app, etc) to be able to do something (write files to S3, send a message to SQS, etc), so you add the matching action to a policy, and things work as expected.

As an example, if there’s an IAM user named Alice, and we want to grant Alice permission to write files to the `acme-photos` S3 bucket, we would make a policy like this (examples are written as CloudFormation resources; even if you don’t know CloudFormation, the pertinent IAM details should be apparent):

```yaml
Alice:
  Type: AWS::IAM::User
  Properties:
    UserName: Alice
    Policies:
      - PolicyName: S3AccessPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowPhotosWrite
              Effect: Allow
              Action: s3:PutObject
              Resource: arn:aws:s3:::acme-photos/*
```

Here there’s a pretty obvious connection between what we want to achieve ("allow Alice to add photos to the `acme-photos` bucket") and the policy definition (`Allow` the `s3:PutObject` action on the `acme-photos` resource).

This was a contrived example for simplicity. IAM [users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) are rarely a good option anymore for granting permissions, and, in cases where they are, inline policies are rarely a best practice. Generally these days, IAM [polices](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) are going to be associated with IAM [roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html). So rather than having an Alice user, you would have something like a `UploadPhotoLambdaRole`. Whether the entity doing work (broadly called a _principal_) is a user, a role, or anything else doesn’t really matter; for a lot of things, it makes intuitive sense how to write a policy that allows a principal to do some required work.

Things get a lot less obvious once IAM policies start dealing with IAM roles. There are three common situations where IAM policies deal with IAM roles, and two of those have the same name because they’re really just two sides of the same coin.

We can start to understand those three cases by looking at least confusing one first. Going back to Alice, if instead of wanting to "allow Alice to add photos to the `acme-photos` bucket", we wanted to "allow Alice to pretend to be the `UploadPhotoLambdaRole`", we take the same approach as before, but use `sts:AssumeRole` instead of `s3:PutObject`.

We create a policy that grants Alice permission to do that action with that specific resource (`Allow` the `sts:AssumeRole` action on the `UploadPhotoLambdaRole` resource). That would look something like this:

```yaml
Alice:
  Type: AWS::IAM::User
  Properties:
    UserName: Alice
    Policies:
      - PolicyName: StsAccessPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowAssumeRoles
              Effect: Allow
              Action: sts:AssumeRole
              Resource: arn:aws:iam::123456789012:role/UploadPhotoLambdaRole
```

This works the same as the earlier example, and probably would never even be that confusing, if not for the other flavor of AssumeRole that trips people up and gets them nervous anytime they see AssumeRole, even when it’s just a vanilla case like this.

And again, Alice is a contrived example, but this pattern happens all the time. Image you have a Lambda function in a **system monitoring** account, and it needs to check the status of some CloudWatch Alarms in the **prod-database** account. No matter how clever you get with the policies of that function’s execution role, it will never be able to hit the CloudWatch Alarms API in the **prod-database** account. Instead, during the execution of the Lambda function, it would have to assume a role that exists within that database account to request the desired data.

So for cases where you want some principal to call `AssumeRole`, you build a policy to allow that, just like you would when you want an entity to call `PutObject` for S3, or `SendMessage` for SQS.

If there is one oddity that can cause confusion with this vanilla flavor of `AssumeRole`, it’s that you may expect to make role-related calls to the IAM API, like you would for other role-related actions. For example, if you want to delete a role, that’s the `DeleteRole` action on the IAM API (granted using `iam:DeleteRole`). Why does `AssumeRole` fall under a different umbrella? The STS service is all about providing _temporary_ credentials, and that’s what `AssumeRole` is actually doing. `AssumeRole` can be thought of as a shorthand for `CreateTemporaryKeyAndSecretForRole`.

Consider that at whatever point you’re calling `AssumeRole`, you must have some IAM credentials already. For the Alice example, it might be a long-term IAM user access key (i.e., an _access key ID_ and a _secret access key_). When Alice requests to assume the `UploadPhotoLambdaRole` role, it’s not the case that Alice’s user access key starts acting as the `UploadPhotoLambdaRole`. API requests (or SDK calls) made with Alice’s access key only and always have the permissions that Alice has. Calling `AssumeRole` _with_ Alice’s access key _for_ `UploadPhotoLambdaRole` returns **another** _access key ID_/_secret access key_ pair. You then, with two sets of valid credentials, have the option of making API calls using Alice’s credentials to do Alice things, or using `UploadPhotoLambdaRole`’s credentials, to do photo uploading things (until they expire and you have to call `AssumeRole` again).

So while the application of `AssumeRole` can get a little confusing in your code, creating IAM policies around `sts:AssumeRole` for this basic case really shouldn’t feel any different than making policies for any other AWS service that a given principal needs to interact with. Try not to overcomplicate this case when you’re dealing with it.

But as promised, there is another flavor of `AssumeRole` that ultimately serves a similar function, but does so in a way that feels quite a bit different. This is the second of our three tricky cases.

Something I glossed over above when discussing the `AssumeRole` policy action is that any time you have principal assuming a role, besides needing to grant that principal the permission to make that request, the role that’s _being assumed_ also needs to explicitly state that the principal in question is _allowed to assume it_. So in the example above, giving Alice `Allow` `sts:AssumeRole` for `UploadPhotoLambdaRole` grants Alice permission to _ask_ to assume the role, but it doesn’t necessarily mean to role will say yes.

To get the role to say yes, we add an `AssumeRolePolicyDocument` to the role. This determines which principals are allowed to assume the role.

```yaml
FunctionExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: UploadPhotoLambdaRole
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Action: sts:AssumeRole
          Principal:
            AWS: arn:aws:iam::123456789012:user/Alice
```

So while this `AssumeRolePolicyDocument` also says `sts:AssumeRole`, it’s not related to this role making an API call to `AssumeRole`. It’s actually saying the opposite; it’s saying "here’s who can call `AssumeRole` _on me_". In this example, we have an `AssumeRolePolicyDocument` that is basically the `UploadPhotoLambdaRole` saying, "hey STS, if you get a request from Alice to assume me, that’s allowed and you should generate temporary credentials for her."

In a lot of places throught AWS, this is also referred to as a _trust relationship_ or _trust policy_, since it’s declaring who this role trusts to act on its behalf. Because remember, once Alice gets the temporary credentials from `AssumeRole`, she can pretend to be `UploadPhotoLambdaRole` as much as she wants, so `UploadPhotoLambdaRole` is tusting Alice with all of its power and priviledge.

To help understand this system even better, let’s look at another (less contrived) example with `UploadPhotoLambdaRole`, which is the execution role for a hypothetical Lambda function that uploads photos to S3. It will need the permissions to do that, which we handle with standard IAM policies:

```yaml
FunctionExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: UploadPhotoLambdaRole
    Policies:
      - PolicyName: S3AccessPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowPhotosWrite
              Effect: Allow
              Action: s3:PutObject
              Resource: arn:aws:s3:::acme-photos/*
```

When the AWS Lambda service itself actually executes the function, it will eventually reach some code that calls `PutObject`, maybe with the JavaScript or Python SDK. For that, it will need valid IAM credentials to make that call (an _access key ID_ and a _secret access key_). IAM roles don’t have long-term credentials, so even though we’ve told the Lambda service to use the `UploadPhotoLambdaRole` when it executes this function, that doesn’t actually give it any credentials to do work with. AWS Lambda has the exact same bag of tricks that we do when it comes to creating temporary credentials from a role: it will have to call `AssumeRole`.

Now, AWS could have made it so that any AWS service is able to call `AssumeRole` for any role that exists in AWS, but as part of AWS’s [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/) we, as users of AWS, are responsible for making those sorts of decisions. So just like above, where we had to list Alice as a trusted principal that’s allowed to assume the role, we now need to list the AWS Lambda service itself as a trusted principal. This looks very similar:

```yaml
FunctionExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: UploadPhotoLambdaRole


    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Action: sts:AssumeRole
          Principal:
            Service: lambda.amazonaws.com


    Policies:
      - PolicyName: S3AccessPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowPhotosWrite
              Effect: Allow
              Action: s3:PutObject
              Resource: arn:aws:s3:::acme-photos/*
```

This role now does the two things we need it to do: it has permission to write photos to S3, and it allows AWS Lambda to request temporary credentials to act as this role.

When the Lambda function executes, the AWS Lambda service will call `AssumeRole` and, since it’s a trusted principal, it will be given a set of temporary credentials. Those credentials are added to the Lambda execution’s environment as environment variables, and the AWS SDKs will detect them and use them when making API calls. That’s why you can generally, for example, do things like `s3.put_object` or `s3.send(new PutObjectCommand())` without ever dealing with key IDs and secrets directly in a Lambda function.

(As a thought exercise, imagine what the execution role policies for the CloudWatch Alarms monitor function I talked about earlier would look like. Make sure you understand why `sts:AssumeRole` would show up in that role _twice_, and how each instance is different.)

You will find yourself doing this same sort of thing for many different services: CloudFormation, CodePipeline, CodeBuild, EventBridge events, etc. Pretty much any time you use an AWS service and you’re asking it to do some work _as_ a particular role (or _using_ a particular role, or _with_ a particular role… there are a bunch of different ways to conceptualize it), the service is going to make an `AssumeRole` call to generate those temporary credentials. Thus each role has to declare a trust relationship with any service that’s going to utilize it in this way.

AWS refers to IAM roles that are assumed by services as **service roles**. A service role is simply any role that you create with the intention of being used by some service like Lambda, CodeBuild, or EventBridge. If you create a role that includes a trust relationship with some AWS service, that is probably a service role. For a lot of AWS users, nearly every role you make will be a service role. _Service role_ is just a label given to a certain (very common) application of IAM roles; don’t get confused and think it’s something entirely distinct from any other IAM role when you see the term. You never have to decide, "should I create a regular role or a service role?" It’s all just roles. AWS isn’t great about using the term consistently. For example, Lambda tends to use the term _execution role_, but this again is just a slightly more specific way of referring to a service-role-used-during-Lambda-function-executions.

(A **service-linked role** is somewhat distinct from other roles, in that they are created and managed by AWS, even though they show up in your account. A service-linked role is a service role where AWS is doing the work to manage the permissions that the service associated with the role will need.)

You’ll find that the `AssumeRolePolicyDocument` is a required element of every IAM role you create. This is because there’s no way to use a role’s permissions _without_ calling `AssumeRole`. A role that doesn’t have a trust policy could never be assumed, and a role that can’t be assumed would be useless; credentials for that role would never be generated, so it would never do any work.

Trust relationships aren’t limited to IAM users and AWS services. They could also declare trust for entire AWS accounts, federated access providers, or other IAM roles. The [AWS docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-roles) have even more examples.

Before we leave the topic of IAM trust policies, it’s worth noting that there are actually other polices in AWS that are very similar. In fact, IAM trust policies are a specific type of the more general **resource-based policy**. Resource-based policies, as exemplified by IAM trust policies, are policies that dictate which principals can access an associated resource and what actions those principals can perform on the resource.

With an IAM role, the resource-based policy is defined by the trust policy, and, as in the examples we’ve looked at, determines which principals can perform `AssumeRole` on it (or other actions, though `AssumeRole` tends to be the most common). Many other resources, like S3 buckets, SNS topics, and Lambda functions, also support resource-based policies. The policies created for resources in these other services will list actions related to that service. So a Lambda function’s resource-based policy may declare which principals can `lambda:InvokeFunction` that function, or an SNS topic’s policy may declare which principals can `sns:Publish` to that topic. Which is to say, `sts:AssumeRole` shows up in the resource-based policy for IAM roles (i.e., trust policies) because that’s the action relevant to role resources; resource-based policies for other services don’t deal with `AssumeRole`, so they won’t show up there. I say that only because if you learn about trust policies as your first type of resource-based policy, you may start to think it’s always about roles or assuming roles.

> (This is off topic, but one example that can help tie all these concepts together would be an EventBridge rule that triggers a Lambda function. The rule will be configured to use a particular role, let’s call it `EventRuleRole`. That role will have a policy to grant it permission to invoke the Lambda function (`lambda:invokeFunction`) and also a trust policy to allow the EventBridge service (`events.amazonaws.com`) to call `AssumeRole` on `EventRuleRole`. Once the EventBridge service calls `AssumeRole`, it will get back temporary credentials that are able to request invocation of the Lambda function. While the `EventRuleRole` has permission to ask, the Lambda function doesn’t necessarily need to say yes. In order to make it say yes, a resource-based policy is created on the Lambda function that declares the EventBridge rule (the specific rule, not the entire EventBridge service) as a principal that’s allowed to invoke it. At that point, the AWS Lambda service will run the function using the execution role that was configured, lets call it `LambdaExecRole`. This has the necessary trust relationship that allows the Lambda service to assume it, and any necessary permissions to do its work.)

And just to close the loop, the IAM policies that are created to grant permissions like writing to S3 or sending SNS messages are called **indentity-based policies**, and they only exist in IAM. As we’ve seen, some IAM roles will have both identity-based policies and resource-based policies (trust policies).

(There are some special cases where you don’t need both the identity-based policy on an IAM role and the resource-based policy on the resource being work on to create a working trust relationship, such as when both the principal and the resource are in the same AWS account. I’d strongly recommend understanding the general case (where both are necessary), and then learning when it’s okay to take shortcuts. It’s always okay to be explicit and do both, even if unnecesary.)

```yaml
FunctionExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: UploadPhotoLambdaRole

    # This is a resource-based policy, called a trust policy with the context of IAM
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Action: sts:AssumeRole
          Principal:
            Service: lambda.amazonaws.com

    # This is an identity-based policy
    Policies:
      - PolicyName: S3AccessPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowPhotosWrite
              Effect: Allow
              Action: s3:PutObject
```

So now we get to the third, and probably most confusing, role-within-a-policy scenario, which is `iam:PassRole`.

`iam:PassRole` shows up in IAM policies, just like `s3:PutObject` or `sns:Publish`, but it’s really important to understand that, unlike pretty much every other IAM policy action, it’s not related to an API call. In the [docs](https://docs.aws.amazon.com/service-authorization/latest/reference/list_awsidentityandaccessmanagementiam.html) you can see that `PassRole` is listed as **[permission only]**.

Or, to put it another way, you will never find yourself making any explicit calls to a `PassRole` API or SDK method, like you would with `AssumeRole`.

Instead, `PassRole` comes into play when managing a resource that has a role associated with it (i.e., a service role). For example, as we saw above, when a Lambda function runs, it utilizes an execution role. That role is configured when the Lambda function is initially created (like when using `aws lambda create-function` or the `CreateFunction` API). When an operation is creating (or updating) a resource that has an associated IAM role like this, the `PassRole` permission is used as a security measure. In these cases the `PassRole` permission is **always** checked, even if you don’t notice.

Let’s say that Alice has **only** the `lambda:CreateFunction` permission on her IAM user; she’s not an administrator or anything like that. The only thing she can do is create Lambda functions. Each time she creates a Lambda function, she needs to provide a role ARN to configure the function’s execution role. If we imagine that Alice is trying to create a Lambda function that uses the `UploadPhotoLambdaRole`, we have to consider if Alice is allowed to create functions using that role.

Remember that Alice only has `lambda:CreateFunction` permission. The `UploadPhotoLambdaRole`, on the other hand, has `s3:PutObject` permission. Does it make sense that Alice shuold be able to create a Lambda function that can upload files to S3, when she herself doesn’t have permission to do that? AWS doesn’t think so, at least not by default. In order for Alice to be able to create a Lambda function associated with that role, she needs to be granted explicit permission to _pass_ that role to AWS Lambda. That’s what `PassRole` is used for.

So rather than just a policy like this:

```yaml
Alice:
  Type: AWS::IAM::User
  Properties:
    UserName: Alice
    Policies:
      - PolicyName: LambdaManagementPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowFunctionCreate
              Effect: Allow
              Action: lambda:CreateFunction
              Resource: "*"
```

Alice’s user would be given the extra `iam:PassRole` permission for that specific role:

```yaml
Alice:
  Type: AWS::IAM::User
  Properties:
    UserName: Alice
    Policies:
      - PolicyName: LambdaManagementPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Sid: AllowFunctionCreate
              Effect: Allow
              Action: lambda:CreateFunction
              Resource: "*"

            - Sid: AllowPassRole
              Effect: Allow
              Action: iam:PassRole
              Resource: arn:aws:iam::123456789012:role/UploadPhotoLambdaRole
```

This signifies that whoever is determining what permissions Alice has, has decided that she should be able to create functions with that role, even though it may have permissions she wouldn’t have herself. She couldn’t suddenly start creating functions with any other role, though, because she’s only been granted `PassRole` for that specific role. Nor can she assume the `UploadPhotoLambdaRole` to start using its `PutObject` permissions directly. All she’s allowed to do it instruct AWS Lambda to use that particular role as a service role for new functions.

In my experience, this mostly comes up with roles needing to pass _other_ roles to various AWS services. For example, if you have pipeline action in CodePipeline that deploys a CloudFormation stack or changeset, that CloudFormation operation can take a service role. That role is responsible for all of the work that needs to happen on the CloudFormation stack, like maybe creating and deleting RDS instances or VPC security groups. While the pipeline itself also has a service role, it doesn’t make sense for the pipeline’s role to be granted permission to do all those things, since it’s never dealing with RDS or VPC directly. Really all it needs permission to do is create and update CloudFormation stacks; it needs to be able to call the CloudFormation API, but not the RDS or VPC APIs. But since the pipeline does need to pass that stack management role to CloudFormation as part of the stack update operaton, it is necessary to grant the pipeline role `PassRole` for that stack management role. Something like:

```yaml
PipelineRole:
  Type: AWS::IAM::Role
  Properties:

    # Trust policy allowing CodePipeline to assume this role
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Action: sts:AssumeRole
          Principal:
            Service: codepipeline.amazonaws.com

    Policies:
      - PolicyName: CloudFormationStackManagementPolicy
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            # Allows this role to call UpdateStack on a specific CloudFormation stack
            - Sid: AllowSecurityGroupStackUpdates
              Effect: Allow
              Action: cloudformation:UpdateStack
              Resource: arn:aws:cloudformation:us-east-1:123456789012:stack/acme-security-groups/*

            # Allows this role to pass the SomeOtherCloudFormationRole role
            # as part of the UpdateStack operation
            - Sid: AllowPassRole
              Effect: Allow
              Action: iam:PassRole
              Resource: arn:aws:iam::123456789012:role/SomeOtherCloudFormationRole
```

This creates a role that can be used within CodePipeline that has very limited access, but can pass a much more powerful role to the CloudFormation service when necesary so that it can do the work the pipeline is actually trying to get done. Without the explicit `PassRole` permission being listed, CloudFormation would reject the stack update request being made using the `PipelineRole`.

It is the case that almost any time you have `PassRole` coming into play, the role that’s being passed around will include a trust policy for some AWS service, and that trust policy will include the `sts:AsumeRole` action. But this is sort of just coincidence; if you’re passing a role to a service, it’s very likely you’re passing it because you want that service to assume the role to do some work. So while there is a loose connection, `PassRole` doesn’t have anything do to with an AWS service actually assuming a role as discussed above to get work done. `PassRole` is a security measure on the API call to create or update a resource, and happens when that request is being processed, not when the service is trying to assume the role.

You also can make it pretty far into your AWS journey without ever seeing `PassRole`. That’s because it’s common for people to have administrator priviledges when they are working in the CLI or web Console, and administrators implicitly have permission to `PassRole` for all roles (`*`), so it may be coming into play fairly often without you realizing it. I think this may be the biggest contributor to why `PassRole` is so mysterious. Doing admin click ops, you run into a lot of IAM action pretty quickly, because you have to explicitly create service roles for Lambda functions and things like that to work, regardless of your own permissions. But `PassRole` only comes into play during resource management, and if you never start doing that programatically, roles will keep magically getting passed around every time you need them, and you’re none the wiser that you actually both need and have that permission.

So to summarize:
- `sts:AssumeRole` shows up in identity-pased policies (i.e., IAM policies) associated with a principal (user, role, etc) when _that_ principal needs to be able to assume a role (i.e., request temporary credentials for a role).
- `sts:AssumeRole` shows up in a resource-based policy (i.e., trust policy) of an IAM role when _other_ principals (roles, users, AWS services, etc) should be allowed to assume the role (i.e., get temporary credentials for the role).
- `iam:PassRole` shows up in identity-pased policies associated with a principal (user, role, etc) when _that_ principal needs to be able to pass some (other) role to AWS services during create and update API calls.

### Postscript

A few related thoughts that can further your knownledge in this area.

1. [Prior](https://aws.amazon.com/blogs/security/announcing-an-update-to-iam-role-trust-policy-behavior/) to 30 June, 2022, an IAM role was able to `sts:AssumeRole` on itself, even without listing itself explicitly in its trust policy. Nowadays, if you want to use a role’s temporary credentails to call `sts:AssumeRole` to create additional temporary credentials for that same role using `AssumeRole`, then you are required to explicitly list the role in its own trust policy. This is a fairly rare use case, but if you run into these sort of recursive policies, it’s likely for this.
2. In a bunch of places throughout this post, I talked about "access key IDs and secret access keys pairs". In practice, especially with temporary credentials, there’s often also a session token that is used as part of the credentials. Key+Secret is the lowest common denominator in terms of various types of credentials across AWS, but it’s not always that simple. APIs like `AssumeRole` will return all necessary componenets of a set of credentials, so as you are using different credential-generating methods, what they give you back is a good indicator of how they are expected to be used when making calls to other service APIs.
3. When building policies with `iam:PassRole`, it’s often a good idea to explicitly ensure that the role can only be passed to specific AWS service. So for the example above, where Alice is passing a role as part of creating Lambda functions, we should ensure at the policy level that she really does only pass that role to the Lambda service.
    ```yaml
    Alice:
      Type: AWS::IAM::User
      Properties:
        UserName: Alice
        Policies:
          - PolicyName: LambdaManagementPolicy
            PolicyDocument:
              Version: "2012-10-17"
              Statement:
                - Sid: AllowFunctionCreate
                  Effect: Allow
                  Action: lambda:CreateFunction
                  Resource: "*"

                - Sid: AllowPassRole
                  Effect: Allow
                  Action: iam:PassRole
                  Condition:
                    StringEquals:
                      iam:PassedToService: lambda.amazonaws.com
                  Resource: arn:aws:iam::123456789012:role/UploadPhotoLambdaRole
    ```
    By adding a `Condition` to the policy, we can further limit what sort of permissions Alice has when it comes to passing this role around. When AWS performs the `PassRole` security check, it will include details in the request about the context of the request. Many services will include a [`PassedToService`](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_iam-condition-keys.html#ck_PassedToService) value, so we can build a condition off that. In this case, a `PassRole` check being performed by the AWS Lambda service would include a matching `PassedToService` value, so this policy would apply and be allowed. Checks performed by any other service would not include a matching value, so in those cases Alice’s user would not have a valid policy that allows passing the role to those services, and they will fail.
    
