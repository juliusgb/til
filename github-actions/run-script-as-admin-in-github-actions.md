# Run PowerShell Script as Admin in GitHub Actions on Self-hosted Runners

This came up when I wanted to run a powershell script with elevated `Administrator` privileges in a CI/CD process.

I'm using GitHub Actions for my CI/CD. These Actions run run on self-hosted runners on Windows.
Though it works, running a script as an `Administrator` like that should alert you to proceed with caution.

## The PowerShell Script

The script is a simple one that writes "Hello World" to a text file.
It's stored in the application's git repository.

```powershell
# RunAsAdmin.ps1
$LocalTestDir="C:\var\test"
if (Test-Path $LocalTestDir -PathType Container) {
  Write-Host "$LocalTestDir exists"
}
else {
  New-Item $LocalTestDir -ItemType Directory
}
Set-Content "$LocalTestDir\test.txt" 'Hello World'
Get-Content "$LocalTestDir\test.txt"
```

## GitHub Actions workflow

```yml
name: Run as admin test
on:
  push:
    branches:
    - test-run-as-admin
jobs:
  run-as-admin-test:
  runs-on: [self-hosted, windows]
  steps:
  - name: Checkout repo
    uses: actions/checkout@v2 # repo contains the RunAsAdmin.ps1
  - name: Run as Admin
    run: |
      $myWindowsID=[System.Security.Principal.WindowsIdentity]::GetCurrent()
      echo "myWindowsID=$myWindowsID"
      $myWindowsPrincipal=new-object System.Security.Principal.WindowsPrincipal($myWindowsID)
      echo "myWindowsPrincipal=$myWindowsPrincipal"
      $adminRole=[System.Security.Principal.WindowsBuiltInRole]::Administrator
      if ($myWindowsPrincipal.IsInRole($adminRole)) { echo "is admin" } else { echo "is not admin"; Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$env:GITHUB_WORKSPACE\MyScript.ps1`"" -Verb RunAs; }
  - name: check
    run: dir "C:\var"
```

With GitHub's ephemeral runners, the run environment and context,
including the filesystem on which the job runs, disappears.
The `check` step works because it's within the same `job`.

If you'd like to access `C:\test\test.txt` in another `job`,
one option would be to upload it as an artifact so that it's accessible in another jobs.

## Other useful sources

- Helpful as a base: <https://github.com/atao/PowerShell-Privileges-Escalation>
- API Docs for the `Start-Process: <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process?view=powershell-7.2>
