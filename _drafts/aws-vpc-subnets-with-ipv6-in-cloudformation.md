---
layout: post
title: AWS VPC subnets with IPv6 in CloudFormation
date: 2020-05-23 08:03 -0400
---

```yaml
VPC:
Type: AWS::EC2::VPC
Properties:
  CidrBlock: 10.0.0.0/16
  EnableDnsSupport: true
  EnableDnsHostnames: true
  InstanceTenancy: default
  Tags:
    - Key: Project
      Value: platform.prx.org
    - Key: Environment
      Value: !Ref EnvironmentType
    - Key: Name
      Value: !Sub Platform-${EnvironmentType}
    - Key: 'prx:cloudformation:stack-name'
      Value: !Ref AWS::StackName
    - Key: 'prx:cloudformation:stack-id'
      Value: !Ref AWS::StackId
```
