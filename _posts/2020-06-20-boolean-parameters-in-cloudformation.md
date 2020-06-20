---
layout: post
title: Boolean parameters in CloudFormation
date: 2020-06-20 00:48 -0400
tags:
  - AWS
  - CloudFormation
---

Consider the following [CloudFormation](https://aws.amazon.com/cloudformation/) template:

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Resources:
  MyMediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      ObjectLockEnabled: true
```

It creates an S3 bucket and sets the `ObjectLockEnabled` property, which has a type of `Boolean` in the CloudFormation [spec](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-objectlockenabled). Now imagine that you want to parameterize the `ObjectLockEnabled` property, so that the value can be determined when the CloudFormation stack is launched using a [template parameter](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html).

There is no `Boolean` type for template parameters. The closest thing is a `String` parameter with two boolean-like values in its `AllowedValues` list. Thankfully, CloudFormation resource properties tend to be very good at type coercion. In nearly all cases, if a property expects a numeric `String` (like `"42"`) but you pass in a `Number`, or vice versa, the system will handle things correctly.

This is true of boolean properties as well. A string value of `"true"` or `"false"` will be treated as a boolean value when passed to a `Boolean` resource property. Thus, the template above would also work with a `String` literal value for `ObjectLockEnabled`:

```yaml

Resources:
  MyMediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      ObjectLockEnabled: 'true'
```

This makes it very easy to use a `String` template parameter to control `Boolean` resource properties:

```yaml
# This is a good way of creating boolean template parameters
AWSTemplateFormatVersion: "2010-09-09"

Parameters:
  BucketObjectLockEnabled:
    Type: String
    AllowedValues:
      - true
      - false

Resources:
  MyMediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      ObjectLockEnabled: !Ref BucketObjectLockEnabled
```

It's worth noting that the `AllowedValues` in the previous example are actual YAML `bool` values. They are being coerced to strings in the context of the parameter, and then the string value is being coerced back to a `Boolean` at the resource property. This method is **not** sneakily creating a boolean parameter. Using string literals in the parameter list acts the same way, and perhaps makes it more obvious what is actually going on.

```yaml
# This way works, too
Parameters:
  BucketObjectLockEnabled:
    Type: String
    AllowedValues:
      - 'true'
      - 'false'
```

Interestingly, `AllowedValues` appears to resolve all possible YAML `bool` values to `"true"` and `"false"`. YAML allows for a number of different reserved words to act as bools: `y`, `yes`, `on`, `off`, `no`, etc. You can use any of those values within `AllowedValues`, and if you create a stack in the Console, the dropdown selector will always show `"true"` and `"false"`.

```yaml
# This works as well, but it probably not recommended, given the number of
# conversions the values will go through (yes to true, true to "true", "true"
# to true)
Parameters:
  BucketObjectLockEnabled:
    Type: String
    AllowedValues:
      - yes
      - no
```

The type coercion on the resource properties **cannot** handle all the various forms of YAML booleans. Passing a string like `"yes"` or `"off"` to a `Boolean` property will fail.

```yaml
# This would not work
Parameters:
  BucketObjectLockEnabled:
    Type: String
    AllowedValues:
      - 'yes'
      - 'no'
```

```yaml
# Neither would this
Resources:
  MyMediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      ObjectLockEnabled: 'off'
```

If the idea of the YAML parser and CloudFormation messing with your value types makes you uncomfortable, there is always the option of dealing with values only in their native types. It's much more verbose, and does not appear to be necessary, but it is also a viable option.

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Parameters:
  BucketObjectLockEnabled:
    Type: String
    AllowedValues:
      # Using string literal values for the String parameter. These can be any
      # arbitrary values you want.
      - Yes_please # Probably use something like 'True' or Enabled instead
      - No_thank_you

Conditions:
  # Create a condition that matches on the parameter string value being treated
  # as true/on.
  HasObjectLockEnabled: !Equals [!Ref BucketObjectLockEnabled, Yes_please]

Resources:
  MyMediaBucket:
    Type: AWS::S3::Bucket
    Properties:
      # Conditionally pass the appropriate YAML `bool` value to the resource
      # property.
      ObjectLockEnabled: !If [HasObjectLockEnabled, true, false]
```
