# AWS CLI - Store and Retrieve Json from AWS Parameter Store

Using the AWS CLI, how do I store Json in AWS Parameter Store?
And how do I retrieve it?

Assuming you have the necessary AWS permissions set up:

- open your terminal/command shell
- navigate to where the json file is stored, like `/path/to/config.json`

To store the json, run

```sh
aws ssm put-parameter --name "/path/to/parameter-in-store/parameter-name" --type "String" --value file://config.json
```

To overwrite the parameter, add the `--overwrite` option.

To retrieve the json, run the AWS CLI and pipe the output to `jq`:

```sh
aws ssm get-parameter --name '/path/to/parameter-in-store/parameter-name' | jq -r '.Parameter.Value'
```
