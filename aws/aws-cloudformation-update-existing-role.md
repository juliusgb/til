# Cloudformation - Update existing role

## Context

I used one Cloudformation template to create an IAM role - an InstanceRole with limited permissions.
I wanted to add additional permissions (e.g., being able to write to a particular path in Parameter Store) in another Cloudformation template.

Using nested stacks was not an option because these two templates have their own runs and are part of different build-and-deploy lifecyles.

## Solution

The idea (adapted from https://aws.amazon.com/premiumsupport/knowledge-center/cloudformation-attach-managed-policy/) is to add the Logical ID (not the Physical ID) of the IAM resource.


```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Updates existing IAM role

Parameters:
  ExistingRoleToUpdate:
  Description: Existing role that needs an additional policy
  Type: String
  Default: my-ec2-InstanceRole # logical ID of resource

Resources:
  ExtendedInstanceRolePolicy:
  Type: AWS::IAM::Policy
  Properties:
    PolicyName: "ExtendedInstanceRolePolicy"
    PolicyDocument:
      Version: '2012-10-17'
      Statement:
      - Effect: Allow
        Action:
          - ssm:PutParameter
          - ssm:AddTagsToResource
          - ssm:RemoveTagsFromResource
        Resource: !Sub arn:${AWS::Partition}:ssm:${AWS::Region}:${AWS::AccountId}:parameter/myparam/test-env/*
    Roles:
      - !Ref ExistingInstanceRole

Outputs:
  ExistingRoleToUpdate:
    Value: !Ref ExistingRoleToUpdate
  ExtendedInstanceRolePolicy:
    Value: !Ref ExtendedInstanceRolePolicy
```

## Checking

- In AWS Console -> IAM -> Roles -> Choose `my-ec2-InstanceRole`
- In the section "Permission policies," -> Click the refresh button
- The new policy `ExtendedInstanceRolePolicy` should show up.
