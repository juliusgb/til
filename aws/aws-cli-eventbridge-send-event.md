
# AWS CLI for EventBridge - Send an event to EventBridge

## Send an event to the default EventBus

I'm assuming that permissions are properly setup.

The hardest part was getting the correct Json structure.

- The `Detail` object contains a mixture of Json (the curly brackets are Json) and escaped strings (the keys and values).
- I escaped all the objects, such as `Source`, but that led to an an invalid Json error.

So save the contents below in a Json file (e.g., `myEventForDefaultEventBus.json`)

```json
[
  {
    "DetailType": "My Awesome Event Name",
    "Detail": "{ \"ec2InstanceId\": \"testInstanceId2\", \"exampleJsonObj\": { \"computerName\": \"fromCLI\" } }",
    "Resources": [ ],
    "Source": "my-service-as-source"
  }
]
```

Then go into the directory where this file is saved and run the aws cli with

```sh
aws events put-events --entries file://myEventForDefaultEventBus.json
```

The output looks like:

```json
{
  "FailedEntryCont": 0,
  "Entries": [
    {
      "EventId": "<some-generated-unique-identifier>"
    }
  ]
}
```

## Send an event to custom EventBus

The event above went to the default event bus because no element `EventBusName` is in the Json structure.

To send an event to a custom event bus, I needed to create the event bus, setup the right permissions, and add `EventBusName` element to the event's Json structure (according to [PutEvents syntax](https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutEvents.html#API_PutEvents_RequestSyntax)).


```json
[
  {
    "DetailType": "My Awesome Event Name",
    "Detail": "{ \"ec2InstanceId\": \"testInstanceId2\", \"exampleJsonObj\": { \"computerName\": \"fromCLI\" } }",
    "EventBusName": "my-custom-event-bus"
    "Resources": [ ],
    "Source": "my-service-as-source"
  }
]
```

Then go into the directory where this file is saved and run the aws cli with

```sh
aws events put-events --entries file://myEventForCustomEventBus.json
```

The output also looks like:

```json
{
  "FailedEntryCont": 0,
  "Entries": [
    {
      "EventId": "<some-generated-unique-identifier>"
    }
  ]
}
```

### Create event bus using CloudFormation

The Cloudformation template below does the following:

- Creates a custom event bus and routes all events to CloudWatch.
- Creates policies that allow this accountId to send events to the custom event bus.

```yml
...
Resources:
  MyCustomEventBus:
    Type: 'AWS::Events::EventBus'
    Properties:
      Name: !Sub "${AWS::StackName}-event-bus"
  
  MyCustomEventBusLogGroup:
    Type: 'AWS::Logs::LogGroup'
    UpdateReplacePolicy: Delete
    DeletionPolicy: Delete
    Properties:
      LogGroupName: !Sub '/aws/events/${AWS::StackName}-events'
      RetentionInDays: 7

  ConduitEventBusEventRule:
    Type: 'AWS::Events::Rule'
    Properties:
      Description: 'Send all events to CloudWatch'
      State: 'ENABLED'
      EventBusName: !Ref 'MyCustomEventBus'
      EventPattern:
        source:
          - my-service-as-source
      Targets:
        - Arn: !GetAtt
            - MyCustomEventBusLogGroup
            - Arn
          Id: MyCustomEventBusEventRuleTrigger
```



## References:

- API reference for PutEvents operation - <https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutEvents.html#API_PutEvents_RequestSyntax>
- AWS CLI docs for `put-events`: <https://docs.aws.amazon.com/cli/latest/reference/events/put-events.html>
- Creating a custom event bus with CloudFormation - <https://hands-on.cloud/eventbridge-building-loosely-coupled-event-drivent-serverless-architectures/#creating-you-own-eventbridge-bus>
- Permissions reference - <https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-permissions-reference.html>