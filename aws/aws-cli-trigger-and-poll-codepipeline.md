# AWS CLI - Trigger and poll Codepipeline

Some of the pipelines I work on deploy workloads in AWS using AWS CodePipeline.
In another CI/CD tool, I'd like to use the `aws cli` to trigger a CodePipeline and periodically check if it succeeded or not.

My setup requires:

- The aws cli is present and correctly configured - be that using AWS `access keys` or using an `assume role`.
- Infrastructure-as-Code written in Cloudformation. This can be ported to your tool of choice.

## Set up permissions for CodePipeline

I found it tricky to properly set up permissions for CodePipeline.

- One reason being that CodePipeline doesn't support `resource-level` policies <https://docs.aws.amazon.com/codepipeline/latest/userguide/security_iam_service-with-iam.html#security_iam_service-with-iam-resource-based-policies>
- There's `resource-based` policies `resource-level` policies, `identity-based` policies. Which one of these applies also depends on the operation you want to execute. This table summarises them: <https://docs.aws.amazon.com/codepipeline/latest/userguide/permissions-reference.html>

```yaml
...
Resources:
  MyRoleName:
    Type: 'AWS::IAM::Role'
    Properties:
      RoleName: ...
      ...
      PermissionBoundary: ...
      Policies:
        - PolicyName: codepipeline
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - codepipeline:GetPipeline
                  - codepipeline:StartPipelineExecution
                  - codepipeline:GetPipelineState
                Resource: !Sub arn:aws:codepipeline:${AWS::Region}:${AWS::AccountId}:MyPipelineName
```

## Combining the steps

I need to add 2 more steps:

- Trigger CodePipeline with `aws codepipeline start-pipeline-execution --name MyPipelineName --region eu-central-1`

- Looking at the [json](https://docs.aws.amazon.com/codepipeline/latest/userguide/pipelines-view-cli.html#pipelines-executions-status-cli) returned by running `aws codepipeline get-pipeline-state --name MyPipelineName --region eu-central-1`
use `jq` to find the last stage of the pipeline and the status of the last action.

```powershell
$PipelineExecution = aws codepipeline start-pipeline-execution --name MyPipelineName
...
Do {
  Start-Sleep -Seconds 10
  $codepipelineState = aws codepipeline get-pipeline-state --name MyPipelineName --region eu-central-1
  $codepipelineStatus = jq '.stageStates | .[length-1].actionStates | .[length-1].latestExecution.status' $codepipelineState
} Until ("Succeeded" -eq $codepipelineStatus -OR "Failed" -eq $codepipelineStatus)
Write-Host "AWS Deployment completed."
# TODO: somehow let ci/cd tool know about success or failed so that the it paints the steps green or red
```

## References

- AWS CLI docs for CodePipeline - <https://docs.aws.amazon.com/cli/latest/reference/codepipeline/index.html>

- AWS reference for CodePipeline's IAM Permissions - <https://docs.aws.amazon.com/codepipeline/latest/userguide/permissions-reference.html>

- `jq` docs - <https://stedolan.github.io/jq/manual/>
