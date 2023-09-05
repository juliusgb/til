# Download with WebClient not Invoke-WebRequest

When downloading files in PowerShell, using [.NET's `WebClient`](https://learn.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-7.0) is way faster than using [`Invoke-WebRequest`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.3).

Recently, downloading a 1.5GB file using `Invoke-WebRequest` took about 20 minutes.
For the same file, download takes about 1 minute when using the `WebClient`.

I ran that on an `t3.medium` EC2 instance.
As for the PowerShell version, the snippet below is the version I'm using:

```powershell
# run
PS C:\> $PSVersionTable.PSVersion

#  output
Major  Minor  Build  Revision
-----  -----  -----  --------
5      1      17763  4644
```

How does the command look like for `WebClient`?

```powershell

# what to download and from where
$downloadUrl = "https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.8.1%2B1/OpenJDK17U-jdk_x64_windows_hotspot_17.0.8.1_1.zip"

$Filename = ([IO.Path]::GetFileName($downloadUrl))
$FilePath = Join-Path (pwd) $Filename

$StopWatch = New-Object System.Diagnostics.StopWatch
$StopWatch.Start()

(New-Object System.Net.WebClient).DownloadFile($downloadUrl, $FilePath)

$StopWatch.Stop()
$DurationInMinutes = $StopWatch.Elapsed.TotalMinutes

Write-Host "Download lasted $DurationInMinutes minutes."
```
And for `Invoke-WebRequest`?

```powershell
# what to download and from where
$downloadUrl = "https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.8.1%2B1/OpenJDK17U-jdk_x64_windows_hotspot_17.0.8.1_1.zip"

# where to save it to
$Filename = $([IO.Path]::GetFileName($downloadUrl))
$FilePath = Join-Path (pwd) $Filename

$StopWatch = New-Object System.Diagnostics.StopWatch
$StopWatch.Start()

Invoke-WebRequest $downloadUrl -OutFile $FilePath

$StopWatch.Stop()
$DurationInMinutes = $StopWatch.Elapsed.TotalMinutes

Write-Host "Download lasted $DurationInMinutes minutes."
```

Why is `Invoke-WebRequest` so slow? Let me know when you find out.

Apparently, with `Invoke-Webrequest`,

> the HTTP response stream is buffered into memory, and once the file has been fully loaded- then only it will be flushed to disk. [https://learn.microsoft.com/en-us/answers/questions/1036880/invoke-webrequest-downloads-a-few-kb-and-stops](https://learn.microsoft.com/en-us/answers/questions/1036880/invoke-webrequest-downloads-a-few-kb-and-stops) 

Other _notable_ places where `WebClient` is used:

- The way GitHub downloads binaries for its GitHub Runners: [https://github.com/actions/runner-images/blob/main/images/win/scripts/ImageHelpers/InstallHelpers.ps1#L197](https://github.com/actions/runner-images/blob/main/images/win/scripts/ImageHelpers/InstallHelpers.ps1#L197)
- Downloading the bootstrapping script to install Chocolatey: see the "command" in step 2 at [https://chocolatey.org/install#individual](https://chocolatey.org/install#individual).

