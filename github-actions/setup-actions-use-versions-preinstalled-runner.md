# setup-actions use versions preinstalled on runner

## tl;dr

`setup-actions` uses versions preinstalled on a self-hosted runner. If a version is specified in the GitHub Actions workflow yml file but it's not preinstalled on the self-hosted runner, then GitHub Actions returns an error.

Fix: use a specific sdk version or update the preinstalled versions.

## Details

Recently, while on a self-hosted runner, and I ran into a problem where using `setup-actions` with a `6.0.x` version of `dotnet` returned the error below:

```console
dotnet-install: Adding to current process PATH: "C:\Program Files\dotnet\". Note: This change will not be visible if PowerShell was run as a child process.
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoLogo -Sta -NoProfile -NonInteractive -ExecutionPolicy Unrestricted -Command "& 'C:\actions-runner\_work\_actions\actions\setup-dotnet\v2\externals\install-dotnet.ps1' -Version 6.0.405"
dotnet-install: Note that the intended use of this script is for Continuous Integration (CI) scenarios, where:
dotnet-install: - The SDK needs to be installed without user interaction and without admin rights.
dotnet-install: - The SDK installation doesn't need to persist across multiple CI runs.
dotnet-install: To set up a development environment or to run apps, use installers rather than this script. Visit https://dotnet.microsoft.com/download to get the installer.

dotnet-install: Extracting the archive.
C:\actions-runner\_work\_actions\actions\setup-dotnet\v2\externals\install-dotnet.ps1 : Exception calling 
"ExtractToFile" with "3" argument(s): "Access to the path 'C:\Program Files\dotnet\dotnet.exe' is denied."
At line:1 char:1
+ & 'C:\actions-runner\_work\_actions\actions\setup-dotnet\v2\externals ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [install-dotnet.ps1], MethodInvocationException
    + FullyQualifiedErrorId : UnauthorizedAccessException,install-dotnet.ps1
 
Error: The process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' failed with exit code 1
```
The corresponding workflow yml looked like this:

```yaml
. . .
jobs:
  jobA:
    steps:
      - name: Setup .NET 6.0.x 
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: |
            '6.0.x'
```

`setup-actions` tries to install the latest patch version, say `6.0.410`. But if the runner only has patch version `6.404`, then we see the error above.

## Fixe(s)

One fix is to use a more specific version that's on the runner, such that the yml looks like:

```yaml
. . .
jobs:
  jobA:
    steps:
      - name: Setup .NET 6.0.x 
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: |
            '6.0.404'
```

Another one is to upgrade the self-hosted runner with newer versions of the SDK.
