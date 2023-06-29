# GitHub Actions - save multiline string to output

Trying to write a multi-line string in GitHub Actions output variable `$GITHUB_OUTPUT` returns the error, `Unable to process file command 'output' successfully.`

[GitHub Actions documentation](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#multiline-strings) shows us how to do that.
But, I didn't understand it till I came across this [stackoverflow answer](https://stackoverflow.com/a/74232400) that I reproduce below.

Let the variable `jsonData` hold example json (from another [jq TIL](https://til.juliusgamanyi.com/posts/jq-access-array-index-in-loop)) using the `here-doc` 


```yaml
name: Multiline in outputs

on:
  push:
    branches:
      '**'

jobs:
  test:
    name: debug
    runs-on: [ubuntu]

    steps:
      - name: prepare multiline string
        id: prep-multiline
        run: |
          jsonData=$(cat <<EOF
          {
            "id": 1,
            "name": "James"
          }
          EOF
          )
          EOF=$(dd if=/dev/urandom bs=15 count=1 status=none | base64)
          echo "JsonInfo<<EOF"$'\n'"$jsonData"$'\n'EOF >> $GITHUB_OUTPUT

    - name: use multiline string
      run: |
        echo "JsonInfo is {% raw %} ${{ steps.prep-multiline.outputs.JsonInfo }} {% endraw %}
```

Explanation of lines 24-25

1. Set output with the defined 'name' (in our case `JsonInfo`), and a 'delimiter' that would mark the end of the data: typically it would be a plain EOF but it's strongly recommended that the delimiter is random.
1. Keep reading each line and concatenating it into one input.
1. Once reaching the line consisting of the defined delimiter, stop processing.
This means that another output could start being added.

That `jsonData` can contain the results of a command that returns json.


```yaml
name: Multiline in outputs

on:
  push:
    branches:
      '**'

jobs:
  test:
    name: debug
    runs-on: [ubuntu]

    steps:
      - name: prepare multiline string
        id: prep-multiline
        run: |
          AWS_REGION="us-east-1"
          EC2_NAME="my-ec2"
            jsonData="$(\
              aws ec2 describe-instances \
                --region "$AWS_REGION" \
                --filters "Name=tag:Name,Values=$EC2_NAME" \
                --query 'Reservations[*].Instances[*].{InstanceId: InstanceId, State: State.Name}' \
                --output json
              )"
        EOF=$(dd if=/dev/urandom bs=15 count=1 status=none | base64)
        echo "JsonInfo<<EOF"$'\n'"$jsonData"$'\n'EOF >> $GITHUB_OUTPUT

    - name: use multiline string
      run: |
        echo "JsonInfo is {% raw %} ${{ steps.prep-multiline.outputs.JsonInfo }} {% endraw %}
```
