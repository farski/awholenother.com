---
layout: post
title: S3 CloudTrail trigger for CodePipeline to avoid infinite loops
date: 2024-02-07 09:04 -0500
tags:
  - AWS
  - CloudTrail
  - CodePipeline
  - S3
---

[AWS CodePipeline](https://aws.amazon.com/codepipeline/) does offer [S3 source actions](https://docs.aws.amazon.com/codepipeline/latest/userguide/action-reference-S3.html), which will trigger the pipeline when changes are made to a specific object in a bucket. At some point, I had a pipeline where there was an S3 source action, but the change detection was being handled by an [Events rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) watching for specific [CloudTrails](https://aws.amazon.com/cloudtrail/) [logs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference.html).

I don’t recall if the native S3 source action previously didn’t do change detection, or why I implemented that myself, but somehow I ended up there.

The only tricky bit was dealing with the fact that when the S3 source action ran as part of the pipeline execution, it would perform a `CopyObject` for the object in question. Because I was watching that object for both `PutObject` and `CopyObject` events in CloudTrail, things would get in a loop. The file would be updated (by some expected external means), the Events rule would trigger the pipeline, which would source the object and copy it, which would trigger the rule, which would trigger the pipelines, etc…

To get around that, the Events rule had to filter CloudTrail events where the principal that made the `CopyObject` call was the pipeline’s execution role. With that in place, all other changes would trigger the rule and the pipeline, but the `CopyObject` performed as part of the pipeline’s source action would not trigger another execution.

```yaml
PipelineS3TriggerEventRule:
  Type: AWS::Events::Rule
  Properties:
    Description: >-
      Triggers a CodePipeline when CloudTrail sees changes in S3 on some object
    EventPattern:
      source:
        - aws.s3
      detail-type:
        - AWS API Call via CloudTrail
      detail:
        eventSource:
          - s3.amazonaws.com
        eventName:
          - PutObject
          - CopyObject
        userIdentity:
          sessionContext:
            sessionIssuer:
              arn:
                # CodePipeline does a CopyObject when it pulls in an S3
                # object as part of a Source action, using the pipeline's
                # role. To prevent an infinite loop, those events need to be
                # filtered out
                - anything-but: !GetAtt MyPipelineRole.Arn
        resources:
          ARN: arn:aws:s3:::my-bucket/my-source-object
    State: ENABLED
    Targets:
      - Arn: !Sub arn:aws:codepipeline:${AWS::Region}:${AWS::AccountId}:${MyPipeline}
        Id: my-target-id
        RoleArn: !GetAtt MyEventsRuleRole.Arn
```
