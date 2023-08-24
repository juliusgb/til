# Pass list to bash for processing

Set list in a GitHub Actions workflow.
And pass it to bash for processing, such as [looping through it](../bash/2023-08-24-bash-declare-and-loop-over-array.md).

```yml
name: my workflow
on:
  workflow_dispatch:
  . . .
  jobs:
    jobA:
      # . . .
      steps:
        - name: Print List
          env:
            MY_LIST: '"a","b","c"'
          run: |
            bash .github/scripts/print-list.sh
          shell: bash
```

Then, in bash, loop over the list and do something with it.

```bash
#!/bin/bash

# file: .github/scripts/print-list.sh

# check that env vars have been set
if [[ -z "${MY_LIST:-}" ]]; then
	echo "ERROR: Missing env var MY_LIST"
exit 1
fi

# string processing via https://stackoverflow.com/a/35894538
for i in ${MY_LIST//,/ }
do
    echo "$i"
    # remove first and last quotes: https://stackoverflow.com/a/9733456
    i_without_quotes=$(sed -e 's/^"//' -e 's/"$//' <<<"$i")
    
    echo "$i"
done
```
