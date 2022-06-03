# Run PowerShell Script as Admin on self-hosted Windows Runners

This came up when I wanted to run a powershell script with elevated `Administrator` privileges,
sometimes on my dev latop and sometimes in a CI/CD process.

Running a script as an `Administrator` like that should alert you to proceed with caution.

As for CI/CD, I'm using GitHub Actions that run run on self-hosted runners on Windows.
That means that I need somehow to see script's output in stdout.

## The PowerShell Script

The script executes a command that requires admin rights.
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

[System.IO.Directory]::GetAccessControl("\\.\pipe\docker_engine") | Format-Table
```

## GitHub Actions workflow

The github action checks out the repository to access `RunAsAdmin.ps1`, executes the script, and checks the results.

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
        whoami
        $myWindowsID=[System.Security.Principal.WindowsIdentity]::GetCurrent()
        echo "myWindowsID=$myWindowsID"
        $myWindowsPrincipal=new-object System.Security.Principal.WindowsPrincipal($myWindowsID)
        echo "myWindowsPrincipal=$myWindowsPrincipal"
        $adminRole=[System.Security.Principal.WindowsBuiltInRole]::Administrator
        New-Item -Path "$LocalTestDir\sysout.txt -ItemType File
        if ($myWindowsPrincipal.IsInRole($adminRole)) `
        { 
          echo "is admin" 
        } `
        else { `
          echo "is not admin"; `
          $passThruArgs = '-NoProfile -ExecutionPolicy Bypass -command', '&', "$env:GITHUB_WORKSPACE\RunAsAdmin.ps1", $arg, '*>', "`"$LocalTestDir\sysout.txt`""; `
          Start-Process powershell -Wait -Verb RunAs -ArgumentList $passThruArgs; `
          Get-Content "$LocalTestDir\sysout.txt"; `
        }`
        dir "C:\var"
    - name: check
      run: dir "C:\var"
```

With GitHub's ephemeral runners, the run environment and context,
including the filesystem on which the job runs, disappears.
The `check` step works because it's within the same `job`.

If you'd like to access `C:\test\test.txt` in another `job`,
one option would be to upload it as an artifact so that it's accessible in another jobs.

## Other useful references

- Helpful as a base: <https://github.com/atao/PowerShell-Privileges-Escalation>
- API Docs for the `Start-Process: <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process?view=powershell-7.2>
- Displaying the stdout and stderr of such a script: <https://stackoverflow.com/questions/50765949/redirect-stdout-stderr-from-powershell-script-as-admin-through-start-process>