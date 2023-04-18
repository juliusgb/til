# Upload file to Nexus Repo Manager

I preferred solution 3 listed on [devops schools](https://www.devopsschool.com/blog/how-to-upload-package-to-nexus-and-artifactory-using-curl-and-powershell/) because it builds on other TILs I alread have,
such as [[Powershell - basic auth in Authorization header]].

The tricky part was using the `InFile` option.

```powershell
# prepare basic auth credentials

$user = "User"
$password = "S3cr3t"
$pair = "$($user):$($password)"
$encodedCreds = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($pair))
$basicAuthValue = "Basic $encodedCreds"

$Headers = @{ 
    Authorization = $basicAuthValue 
}

$NexusUrl = "https://nexus.uri.com"
$Repository = "my-raw-repo"
$AssetGroupName = "/path/to/app/v1.0.3"
$file = "myapp.zip"
$Uri = "$NexusUrl/repository/$Repository/$file"

Invoke-RestMethod -UseBasicParsing -Headers $headers -Method PUT -InFile $file -Uri $Uri
```