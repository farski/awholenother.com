---
layout: post
title: Optional comma-delimited list parameters in CloudFormation
date: 2020-06-19 19:47 -0400
tags:
  - AWS
  - CloudFormation
---

When a resource property in a [CloudFormation](https://aws.amazon.com/cloudformation/) template, such as the [`SubjectAlternativeNames`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html#cfn-certificatemanager-certificate-subjectalternativenames) property of a an [`AWS::CertificateManager::Certificate`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html#cfn-certificatemanager-certificate-subjectalternativenames), takes a list of strings, you can use a template parameter with the `CommaDelimitedList` type to pass the list of values in. CloudFormation will expand the parameter into an array when it is passed to the property with [`Ref`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html).

Most properties that expect a list will reject empty an `CommaDelimitedList` parameter (i.e., if the parameter is passed to the property, it **must** include one or more elements). So simply defining, e.g., `SubjectAlternativeNames: !Ref CertificateSubjectAlternativeNames` wouldn’t work when the parameter is empty. For these optional parameters, the property must conditionally get passed [`!Ref AWS::NoValue`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html#cfn-pseudo-param-novalue) when the parameter is excluded, so that the property is ignored entirely by CloudFormation.

Within a template [condition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html), there’s no direct way to test the presence or size of a `CommaDelimitedList` parameter. In order to create a condition that can be used to properly switch between the list and `AWS::NoValue`, you must first call [`Fn::Join`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-join.html) on the list. You can call `Join` on a `CommaDelimitedList` parameter even if it’s blank. Joining an empty `CommaDelimitedList` will result in an empty string, which is something that the condition can test for with [`Fn::Equals`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-conditions.html#intrinsic-function-reference-conditions-equals).

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Parameters:
  # This is a `CommaDelimitedList` parameter that should be optional.
  CertificateSubjectAlternativeNames:
    Type: CommaDelimitedList
    Description: >-
      A comma-delimited list of alternative domain names to add to the
      certificate (e.g., "www.example.com,beta.example.com,app.example.com")

Conditions:
  # In order to check if any values were included in the template parameter,
  # the parameter is joined. When no values were included, the join operation
  # will result in an empty string. Using the `Fn::Not` condition function
  # along with `Fn::Equals` creates a condition that is FALSE when the
  # parameter is left blank, and TRUE when it contains at least one item.
  HasCertificateSubjectAlternativeNames: !Not [!Equals [!Join ['', !Ref CertificateSubjectAlternativeNames], '']]

Resources:
  Certificate:
    Type: AWS::CertificateManager::Certificate
    Properties:
      DomainName: !Ref CertificateDomainName
      # The `SubjectAlternativeNames` property expects a list of strings. If
      # any values have been added to the template parameter, it should be
      # passed to the property. If the template parameter is not defined, the
      # `AWS::NoValue` pseudo parameter needs to be passed in instead, so that
      # the property doesn't end up with an empty list.
      #
      # Note: When using the inline YAML array syntax, AWS::NoValue must be
      # quoted, or CloudFormation will fail to parse the template.
      SubjectAlternativeNames: !If [HasCertificateSubjectAlternativeNames, !Ref CertificateSubjectAlternativeNames, !Ref 'AWS::NoValue']
```
