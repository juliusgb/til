# Grant user access to Docker Pipe

Goals:

- able to run `docker` commands without getting `Administrator` privileges.
- Run `docker` commands in a GitHub Action that's running on Self-hosted runners.

When running `docker build` as part of a step in a GitHub action, we're greeted with the error

> error during connect: In the default daemon configuration on Windows, the docker client must be run with elevated privileges to connect.: Post http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/images/create?fromImage=mcr.microsoft.com%2Fwindows%2Fnanoserver&tag=1809: open //./pipe/docker_engine: The system cannot find the file specified.

## The Fix

Grant the user under which the GitHub Action runs access only to the Docker Pipe
as in <https://github.com/juliusgb/til/blob/main/powershell/grant-access-to-docker-pipe.md>
and not to the whole machine.

```powershell
# GrantAccessToDockerPipe.ps1
# account that's to be granted access. When on laptop, I use juliusg.
$account="NT AUTHORITY\NetworkService" 
$npipe = "\\.\pipe\docker_engine"
$dInfo = New-Object "System.IO.DirectoryInfo" -ArgumentList $npipe
$dSec = $dInfo.GetAccessControl()
$fullControl =[System.Security.AccessControl.FileSystemRights]::FullControl
$allow =[System.Security.AccessControl.AccessControlType]::Allow
$rule = New-Object "System.Security.AccessControl.FileSystemAccessRule" -ArgumentList $account,$fullControl,$allow
$dSec.AddAccessRule($rule)
$dInfo.SetAccessControl($dSec)
```

Doing that as part of preparing the runner means I don't need to think about it anymore.

- After the runner starts, `GrantAccessToDockerPipe.ps1` is executed.
- Now the user of the runner has access to the Docker Pipe and can execute docker commands.

## The Workflow

The github action checks out the repository to access a couple of files:

```yml
name: Docker build and run
on:
  push:
    branches:
    - test-run-as-admin
jobs:
  run-as-admin-test:
  runs-on: [self-hosted, windows]
  steps:
    - name: Checkout repo
      uses: actions/checkout@v2 # repo contains the docker file
    - name: Display User
      run: |
        whoami
    - name: Run docker build
      run: docker build --file "$env:GITHUB_WORKSPACE\Dockerfile-Windows" --tag localhost:5000/mydockerapp:0.0.1
```

## References

- Grant user access to docker pipe <https://github.com/tfenster/dockeraccesshelper> based on <https://www.axians-infoma.de/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/>
- About the `Network Service` account: https://docs.microsoft.com/en-us/windows/win32/services/networkservice-account