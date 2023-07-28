---
layout: post
title: Wrapping CloudFormation parameter list items with additional data
date: 2023-07-28 09:36 -0400
tags:
  - AWS
  - CloudFormation
---

If you have a comma-delimited list CloudFormation stack parameter that, for example, takes a list of S3 bucket names…

```yaml
Parameters:
  DestinationBucketNames:
    Type: CommaDelimitedList
```

you may find that somewhere in the template you need to augment those values with a prefix or a suffix. A common situation would be converting a bucket name (e.g., `MyBuckt`) into a complete ARN (e.g., `arn:aws:s3:::MyBucket`), as would be required in an IAM policy.

Getting a little bit clever with string functions makes this possible. To add a prefix before each value (`arn:aws:s3:::`)

```yaml
- PolicyDocument:
    Statement:
      - Action: s3:ListBucketMultipartUploads
        Effect: Allow
        Resource: !Split
          - ','
          - !Sub
            - 'arn:aws:s3:::${inner}'
            - inner: !Join
                - ',arn:aws:s3:::'
                - Ref: DestinationBucketNames
```

And to add a prefix and a suffix (`/*`):

```yaml
- PolicyDocument:
    Statement:
      - Action: s3:GetObject
        Effect: Allow
        Resource: !Split
          - ','
          - !Sub
            - 'arn:aws:s3:::${inner}/*'
            - inner: !Join
                - '/*,arn:aws:s3:::'
                - Ref: DestinationBucketNames
```

Both cases are doing basically the same thing:

1. Take the parameter list (e.g., `[MyBucket, AnotherBucket, LastBucket]`)
1. Join them into a string using a delimiter that includes a comma (`,`) _and_ the prefix or suffix you’re adding (resulting in `"MyBucket/*,arn:aws:s3:::AnotherBucket/*,arn:aws:s3:::LastBucket"`)
1. Use `!Sub` to add the prefix to the first item in the list and the suffix to the last item (`"arn:aws:s3:::MyBucket/*,arn:aws:s3:::AnotherBucket/*,arn:aws:s3:::LastBucket/*"`)
1. Split the final string on the comma to end up back with a list that the template needs (`[arn:aws:s3:::MyBucket/*, arn:aws:s3:::AnotherBucket/*, arn:aws:s3:::LastBucket/*]`)
