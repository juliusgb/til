# Basic auth in Authorization header

Given a username and password, how do I add them as encoded values in a HTTP header when making a webservice call with `Invoke-RestMethod` ?

```powershell
$user = "User"
$password = "S3cr3t"
$pair = "$($user):$($password)"

$encodedCreds = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($pair))

$basicAuthValue = "Basic $encodedCreds"

$Headers = @{ 
    Authorization = $basicAuthValue 
}
$Uri = "https://example.com/path"
Invoke-RestMethod -UseBasicParsing -Headers $headers -Method GET -Uri $Uri
```

## References

- Stackoverflow: https://stackoverflow.com/a/27951845