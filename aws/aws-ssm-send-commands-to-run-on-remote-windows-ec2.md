# AWS SSM - Send commands to run on remote Windows EC2 Instance

TIL at <https://til.juliusgamanyi.com/posts/aws-ssm-send-commands-to-run-on-remote-windows-ec2/>

Aim: Execute commands _from_ my local machine (even a CI machine) on a remote Windows EC2 instance.
In other words, send a bunch of commands that get executed on a remote EC2 instance. And get back the status of the execution and the its output.
This relies on the [RunCommand feature](https://docs.aws.amazon.com/systems-manager/latest/userguide/running-commands.html) of AWS Session Manager.

## Prerequisites

A few prerequisites are: 

- EC2 instance is running windows;
- The instance profile has role that allows connecting to SSM like `arn:${AWS::Partition}:iam::aws:policy/AmazonSSMManagedInstanceCore`
- aws cli installed and configured locally. I set mine to use a specific profiles. If you don't use profiles, then leave off the profile part of the command.

## Workflow

I use the [`aws ssm send-command`](https://docs.aws.amazon.com/cli/latest/reference/ssm/send-command.html) to execute a command on the EC2 instance. This queues the command to run on the instance. 
To know the if the command was executed and what the output was, I run the [`aws ssm get-command-invocation`](https://docs.aws.amazon.com/cli/latest/reference/ssm/get-command-invocation.html)


### What's the spec for that document?


What the document is, what it allows us to send is its spec or API. 
Since the target of my commands is a Windows instance, I'll use PowerShell.

To see its spec, run `aws ssm describe-document --name "AWS-RunPowerShellScript"`. 
The json output is below:

```json
{
    "Document": {
        "Hash": "2142e42a19e0955cc09e43600bf2e633df1917b69d2be9693737dfd62e0fdf61",
        "HashType": "Sha256",
        "Name": "AWS-RunPowerShellScript",
        "Owner": "Amazon",
        "CreatedDate": "2017-08-30T23:34:35.400000+02:00",
        "Status": "Active",
        "DocumentVersion": "1",
        "Description": "Run a PowerShell script or specify the paths to scripts to run.",
        "Parameters": [
            {
                "Name": "commands",
                "Type": "StringList",
                "Description": "(Required) Specify the commands to run or the paths to existing scripts on the instance."
            },
            {
                "Name": "workingDirectory",
                "Type": "String",
                "Description": "(Optional) The path to the working directory on your instance.",
                "DefaultValue": ""
            },
            {
                "Name": "executionTimeout",
                "Type": "String",
                "Description": "(Optional) The time in seconds for a command to be completed before it is considered to have failed. Default is 3600 (1 hour). Maximum is 172800 (48 hours).",
                "DefaultValue": "3600"
            }
        ],
        "PlatformTypes": [
            "Windows",
            "Linux"
        ],
        "DocumentType": "Command",
        "SchemaVersion": "1.2",
        "LatestVersion": "1",
        "DefaultVersion": "1",
        "DocumentFormat": "JSON",
        "Tags": []
    }
}
```

From the `parameters` block, we see that we need to send `commands`. The other 2 (`workingDirectory`, `executionTimeout`) are useful but not important for this TIL.
 
### For simple commands, string might be enough

For a simple command like print all environment variables (`dir env:`), a string might be enough in the `commands` parameter.

It would look like this snippet below (see line 4):

```sh
aws ssm send-command \
--instance-ids "i-06c8b03dd27afb8af" \
--document-name "AWS-RunPowerShellScript" \
--parameters commands="dir env:" \
--output json \
--profile dev-a
```

From the output, we're interested in the `CommandId` (line 3) and the `Status` (line 18). The output looks like this:

```json
{
    "Command": {
        "CommandId": "44dd9ec3-a7ea-469d-99ca-441352aa7b97",
        "DocumentName": "AWS-RunPowerShellScript",
        "DocumentVersion": "$DEFAULT",
        "Comment": "",
        "ExpiresAfter": "2024-03-01T20:48:08.161000+01:00",
        "Parameters": {
            "commands": [
                "dir env:"
            ]
        },
        "InstanceIds": [
            "i-06c8b03dd27afb8af"
        ],
        "Targets": [],
        "RequestedDateTime": "2024-03-01T20:48:08.161000+01:00",
        "Status": "Pending",
        "StatusDetails": "Pending",
        "OutputS3Region": "us-east-1",
        "OutputS3BucketName": "",
        "OutputS3KeyPrefix": "",
        "MaxConcurrency": "50",
        "MaxErrors": "0",
        "TargetCount": 1,
        "CompletedCount": 0,
        "ErrorCount": 0,
        "DeliveryTimedOutCount": 0,
        "ServiceRole": "",
        "NotificationConfig": {
            "NotificationArn": "",
            "NotificationEvents": [],
            "NotificationType": ""
        },
        "CloudWatchOutputConfig": {
            "CloudWatchLogGroupName": "",
            "CloudWatchOutputEnabled": false
        }
    }
}
```

If I'm in a CI environment, then I'm only interested in the `CommandId` because the subsequent calls use the `CommandId` as the query string.

```sh
command_id=$( \
	aws ssm send-command \
	--instance-ids "i-06c8b03dd27afb8af" \
	--document-name "AWS-RunPowerShellScript" \
	--parameters commands="dir env:" \
	--query "Command.CommandId"  \
	--output text  \
	--profile ci
)
echo "CommandId is $command_id"

# output is CommandId is 44dd9ec3-a7ea-469d-99ca-441352aa7b97
```


### For more complex commands, prefer an array or json in the command parameter

If I have a more complex command or one that has special characters, then using a json object (like this [stackoverflow answer](https://stackoverflow.com/a/75089861)) in the `commands` parameter is preferable to using a string.

Say, you'd like to see the host and domain that an EC2 instance is joined to.
Executing `[System.Net.Dns]::GetHostByName($env:computerName)` on the instance itself returns a decent output.

Executing the same

```sh
aws ssm send-command \
--instance-ids "i-06c8b03dd27afb8af" \
--document-name "AWS-RunPowerShellScript" \
--parameters commands="[System.Net.Dns]::GetHostByName($env:computerName)" \
--output json \
--profile dev-a
```

returns this error

```console
Error parsing parameter '--parameters': Expected: ',', received: ':' for input:
commands=[System.Net.Dns]::GetHostByName(:computerName)
```


Switching to array-style would look like:

```sh
aws ssm send-command \
--instance-ids "i-06c8b03dd27afb8af" \
--document-name "AWS-RunPowerShellScript" \
--parameters commands='["[System.Net.Dns]::GetHostByName","$env:computerName"]' \
--output json \
--profile dev-a
```


### Status and output of the command

Running the `aws ssm get-command-invocation` below, gives 

```sh
aws ssm get-command-invocation \
--command-id "5f7d72c7-9cf4-4a4d-a7ed-fefd06d22f8c" \
--instance-id "i-06c8b03dd27afb8af" \
--output json \
--profile dev
```

The output looks like

```json
{
    "CommandId": "5f7d72c7-9cf4-4a4d-a7ed-fefd06d22f8c",
    "InstanceId": "i-06c8b03dd27afb8af",
    "Comment": "",
    "DocumentName": "AWS-RunPowerShellScript",
    "DocumentVersion": "$DEFAULT",
    "PluginName": "aws:runPowerShellScript",
    "ResponseCode": 0,
    "ExecutionStartDateTime": "2024-03-01T20:08:56.792Z",
    "ExecutionElapsedTime": "PT2.44S",
    "ExecutionEndDateTime": "2024-03-01T20:08:58.792Z",
    "Status": "Success",
    "StatusDetails": "Success",
    "StandardOutputContent": "\r\nOverloadDefinitions                                           \r\n-------------------                                           \r\nstatic System.Net.IPHostEntry GetHostByName(string hostName)  \r\n                                                              \r\nMVP-A12345F\r\n\r\n\r\n",
    "StandardOutputUrl": "",
    "StandardErrorContent": "",
    "StandardErrorUrl": "",
    "CloudWatchOutputConfig": {
        "CloudWatchLogGroupName": "",
        "CloudWatchOutputEnabled": false
    }
}
```

In CI machines, I'd run

```sh
aws ssm send-command \
--instance-ids "i-06c8b03dd27afb8af" \
--document-name "AWS-RunPowerShellScript" \
--parameters commands='["[System.Net.Dns]::GetHostByName","$env:computerName"]' \
--query "Command.CommandId"  \
--output text \
--profile ci
```

## Initial attempt that didn't work

Because I started this on a Linux/Mac machine, I thought using `AWS-RunShellScript` would be sufficient.
But that failed with the error `An error occurred (UnsupportedPlatformType) when calling the SendCommand operation: Cannot perform operation for instance id i-xxx of platform type Windows`.

Running the `aws ssm describe-document --name "AWS-RunShellScript"` gives the output below, where the `PlatformTypes` excludes `Windows`. Duh!

```json
{
    "Document": {
        "Hash": "99749de5e62f71e5ebe9a55c2321e2c394796afe7208cff048696541e6f6771e",
        "HashType": "Sha256",
        "Name": "AWS-RunShellScript",
        "Owner": "Amazon",
        "CreatedDate": "2017-08-30T23:33:44.432000+02:00",
        "Status": "Active",
        "DocumentVersion": "1",
        "Description": "Run a shell script or specify the commands to run.",
        "Parameters": [
            {
                "Name": "commands",
                "Type": "StringList",
                "Description": "(Required) Specify a shell script or a command to run."
            },
            {
                "Name": "workingDirectory",
                "Type": "String",
                "Description": "(Optional) The path to the working directory on your instance.",
                "DefaultValue": ""
            },
            {
                "Name": "executionTimeout",
                "Type": "String",
                "Description": "(Optional) The time in seconds for a command to complete before it is considered to have failed. Default is 3600 (1 hour). Maximum is 172800 (48 hours).",
                "DefaultValue": "3600"
            }
        ],
        "PlatformTypes": [
            "Linux",
            "MacOS"
        ],
        "DocumentType": "Command",
        "SchemaVersion": "1.2",
        "LatestVersion": "1",
        "DefaultVersion": "1",
        "DocumentFormat": "JSON",
        "Tags": []
    }
}
```
