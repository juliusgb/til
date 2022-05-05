# Troubleshoot: Networking issues

I spent most of this week on dealing with these 2 exceptions:

- `Exception calling "DownloadFile" with "2" argument(s): "An exception occurred during a WebClient request."`
- `Exception calling "DownloadFile" with "2" argument(s): "The underlying connection was closed: An unexpected error occured on a send."`

The trigger was:

```powershell
$Url = 'https://community.chocolatey.org/api/v2/package/chocholatey/1.1.0'
$file = Join-Path $pwd "chocoDownload.zip"
(New-Object System.Net.WebClient).DownloadFile($Url, $file)
```

That `$Url` was on the safe/accept list. So why does error occur?

## TL;dr

Turns out that when hitting the url (on the allow/safe list),
we get back a redirect pointing to another url that's not on the allow/safe list.

## The details

- What methods are used: `Invoke-WebRequest` or `New-Object System.Net.WebClient`?
- Try with `curl`
- Does the URL redirect somewhere? If so, where? Use the `Get-UrlRedirect` Function from <https://github.com/juliusgb/utils/blob/main/powershell/CustomHelperUtils/UrlHelpers.ps1#L20-L33>
- Does the DNS name have an IP address? (Use nslookup)
- Does the request reach the target ? (Use tracert)
- Does the request go through a Proxy? Show proxy settings

  - Show proxy settings: `PS> Get-ItemProperty -Path "Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"`
  - Another way WebProxy Settings: `[System.Net.WebProxy]::GetDefaultProxy()`