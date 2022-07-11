
# AWS CLI for EventBridge - Send an event to EventBridge

The hardest part was getting the correct Json structure.

- The `Detail` object contains a mixture of Json (the curly brackets are Json) and escaped strings (the keys and values).
- I escaped all the objects, such as `Source`, but that led to an an invalid Json error.

So save the contents below in a Json file (e.g., `MyEvent.json`)

```json
[
  {
    "Source": "my-source-service",
    "Detail": "{ \"ec2InstanceId\": \"testInstanceId2\", \"exampleJsonObj\": { \"computerName\": \"fromCLI\" } }",
    "Resources": [ ],
    "DetailType": "My Awesome Event Name"
  }
]
```

Then go into the directory where this file is saved and run the aws cli with `aws events put-events --entries file://myEvent.json`

References:
- CLI docs: [https://docs.aws.amazon.com/cli/latest/reference/events/put-events.html](https://docs.aws.amazon.com/cli/latest/reference/events/put-events.html)