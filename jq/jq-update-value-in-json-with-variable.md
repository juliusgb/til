# jq - Update value in json with variable

I use [jq](https://stedolan.github.io/jq) for reading and filtering json on the command line.

Given the json below, I'd like to update the value of `b` to be `30`.
jq offers the [_update assignment_](https://stedolan.github.io/jq/manual/#Assignment), `|=`, for this.

```json
{
  "a": {
    "b": 10
   },
  "c": 20
}
```

Try out the snippet at `https://jqplay.org/s/t39OXPDrll-`

Doing the same with that data saved to file, say `myData.json` and PowerShell and passing the new value as a jq variable would look like this:

```json
{
  "a": {
    "b": 10
   },
  "c": 20
}
```

In the command line, run the following:

```powershell
# retrieve b's value
C:\opt\jq-win64.exe '.a.b' myData.json

# set 30 to a powershell variable
$newValue = 30

# assign newValue to jq variable 'jqVarToUse' and pass it to the assigment operator
C:\opt\jq-win64.exe --arg jqVarToUse $newValue '.a.b |= $jqVarToUse' myData.json

```

jq gives the following output, and **does not modify** the original file.

```json
{
  "a": {
    "b": "30"
   },
  "c": 20
}

```