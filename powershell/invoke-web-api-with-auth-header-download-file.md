# Invoke Web API with Auth token for file downloads

How do I download a remote file via an API that also expects an auth Token?

```powershell
$DownloadedFile = "C:\tmp\filename.zip";

$Url = "https://path/"

$headers = @{
    'Accept' = 'application/octet-stream';
    Authorization = "Bearer $TOKEN";
};

Invoke-WebRequest -UseBasicParsing -Uri $Url -Headers $headers -Outfile $DownloadedFile
```

## Alternative: use .NET WebClient

Another way is to .NET `WebClient` instead of Invoke-Webrequest for downloading files.

I don't remember where I read that `Invoke-Webrequest` copies all of the data into memory before writing it to disk which makes it very slow and resource intensive for downloading files.

An example of where the `Webclient` is wrapped in a callabe Powershell function is in <https://github.com/actions/runner-images/blob/main/images/win/scripts/ImageHelpers/InstallHelpers.ps1#L172>
