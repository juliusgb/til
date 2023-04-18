# Read multiple values and assign to variables in bash

Sometimes I need to read multiple string values from json, and save them into their own variables as a pre-processing step.

Assigning multiple variables in Bash using [process substitution, i.e., `<(command)`](https://en.wikipedia.org/wiki/Process_substitution) and using `join` with an empty string saved the day. 

Let the variable `jsonData` hold example json (from another [jq TIL](https://til.juliusgamanyi.com/posts/jq-access-array-index-in-loop)) using the `here-doc` 

```bash
# load json data
jsonData=$(cat <<EOF
  {
    "id": 1,
    "name": "James"
  }
  EOF
)

# assign vars using process substitution
read -r var1 var2 < <(echo $jsonData | jq -r '[ .id, .name] | join(" ")')
echo "id=$var1 is called $var2"
```

## References

- Stackoverflow: https://stackoverflow.com/a/43293515
- Assigning multiple variables in Bash using process substitution: https://www.baeldung.com/linux/bash-multiple-variable-assignment
- Difference between process substitution (`<(command)`) and here-document structure (`<<`): https://askubuntu.com/questions/678915/whats-the-difference-between-and-in-bash/678919#678919 and https://www.baeldung.com/linux/bash-multiple-variable-assignment and https://www.baeldung.com/linux/heredoc-herestring#here-string
