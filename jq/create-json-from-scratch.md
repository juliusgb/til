# Create json from scratch

How do I create a json structure from scratch using only [`jq`](https://stedolan.github.io/jq/download/)?


The command line option `--arg name value`
> passes a value to the jq program as a predefined variable...
> so `--arg foo 123` will bind `$foo` to `"123"` - https://stedolan.github.io/jq/manual/

These `named arguments` are also available from `$ARGS.named`. Found via https://stackoverflow.com/a/59769162

The snippet below creates a json structure that you can pipe to a file.
It also prints the contents of `$ARGS` to show what arguments are `named` and which are `positional`.
`jq` also support linefeeds, so we can make the json object look pretty.

```sh
$reponame="awesome-repo"
$branchname="main"

jq -n '
	{
		github_meta: {
			reponame: $reponame,
			branch_name: $branchname
		},
		feature: {},
		all: $ARGS
	}' \
  --arg reponame "$reponame" \
  --arg branchname "$branchname" \

# output
{
  "github_meta": {
    "reponame": "awesome-repo",
    "branch_name": "main"
  },
  "feature": {},
  "all": {
    "positional": [],
    "named": {
      "reponame": "awesome-repo",
      "branchname": "main"
    }
  }
}
```
