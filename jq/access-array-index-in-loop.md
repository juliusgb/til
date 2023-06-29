# jq - Access array index in a loop

If your json contains a list of objects, `jq` allows you to access such elements using an [array index filter](https://stedolan.github.io/jq/manual/#Basicfilters) `jq '.[index]' someData.json`. But the value `index` is static, say `jq '.[2]' someData.json`.

How can I access the element at position `index` when the value of `index` is dynamic, such as when in loops?

What I want to do is, when in a loop, call `jq` to access an element at some index that I specify.

In pseudocode, it'd look something like this :

```sh
array=[ {"name": "a"}, {"name": "b"}, {"name": "c"} ]
for (( i = 0; i < $array.length; i++ ))
do
    element = jq '[$i]' $array
    echo "element = $element"
done
```

## What worked - using `--argjson`

For that to work, we need to be in the context of `jq`, where:

1. we declare a variable and assign it a value;
2. and then use that variable as an argument for the array index.

That means:

- Using `--argjson` to assign a numeric variable that's passed to `jq` as the index of the array. It has the form: `--argjson myIndex $i '.[$myIndex]'`
  - The first part, `--argjson myIndex $i`, defines variable `myIndex` and assigns the value of `$i` to it.
  - The second part, `'.[$myIndex]'`, passes the argument `$myIndex` to the [array index filter](https://stedolan.github.io/jq/manual/#Basicfilters).

- Using `--raw-output` to get the result that's not "formatted as a JSON string with quotes".

Let's set the `bash` variable `jsonData` to hold example json (via [Stackoverflow](https://stackoverflow.com/a/43373520))

```sh
# load json data
jsonData=$(cat <<EOF
  [
    {
      "id": 1,
      "name": "One"
    },
    {
      "id": 2,
      "name": "Two" 
    },
    {
      "id": 3,
      "name": "Three" 
    }
  ]
  EOF
)

numberOfItems=$(echo $jsonData | jq --raw-output length)

# loop over the list in json
for (( i=0; i<numberOfItems; i++ ))
do
    echo "At index, $i"
    # assign a numerical variable that's passed to jq as the array index
    name=$( echo $jsonData | jq --raw-output --argjson myIndex $i '.[$myIndex].name' )
    echo "Name=$name"
done
```

## What didn't work - using variables not in jq context

Using the `i` in the loop directly as the index of the array didn't work.

```sh
jsonData=$(cat <<EOF
  [
    {
      "id": 1,
      "name": "One"
    },
    {
      "id": 2,
      "name": "Two" 
    },
    {
      "id": 3,
      "name": "Three" 
    }
  ]
  EOF
)

numberOfItems = $(echo $jsonData | jq --raw-output length)

# loop over the list in json
for (( i=0; i<numberOfItems; i++ ))
do
    echo "At index, $i"
    # passing i directly doesn't work
    name=$( echo $jsonData | jq --raw-output '.[$i].name' )
    echo "name=$name"
done
```

That returned error

```sh
jq: error: $i is not defined at <top-level>, line 1:
.[$i].name  
jq: 1 compile error
```

Why? Although `i` is defined in the shell script, it's not defined in the `jq`'s context. So `jq` has no idea about `ì`.

## What didn't work - using `--arg`

Using `--arg name value` also didn't work.

```sh
jsonData=$(cat <<EOF
  [
    {
      "id": 1,
      "name": "One"
    },
    {
      "id": 2,
      "name": "Two" 
    },
    {
      "id": 3,
      "name": "Three" 
    }
  ]
  EOF
)

numberOfItems = $(echo $jsonData | jq --raw-output length)

# loop over the list in json
for (( i=0; i<numberOfItems; i++ ))
do
    echo "At index, $i"
    # using --arg doesn't work
    name=$( echo $jsonData | jq --raw-output --arg myIndex $i '.[$myIndex].name' )
    echo "name=$name"
done
```

I'm assigning `i` to variable `myIndex` so as to pass it to jq. But that fails with

```sh
jq: error (at jsonData.json:2): Cannot index array with string "0"
```

Why? Because `--arg` assigns a variable as a `String`, but `jq`'s array index must be a number.

## References

- `jq` documentation: <https://stedolan.github.io/jq/manual/#Invokingjq>

- Stackoverflow: <https://stackoverflow.com/questions/52609226/how-i-pass-argument-as-index-to-jq>
