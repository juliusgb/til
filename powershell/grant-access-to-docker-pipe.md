# Grant user access to Docker Pipe without Admin rights

Aim:

- able to run `docker` commands without getting `Administrator` privileges.
- Run `docker` commands in a GitHub Action that's running on Self-hosted runners. See https://github.com/juliusgb/til/blob/main/github-actions/run-docker-commands.md

When running `docker build` on local dev machine, we're greeted with the error:

> error during connect: In the default daemon configuration on Windows, the docker client must be run with elevated privileges to connect.: Post http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/images/create?fromImage=mcr.microsoft.com%2Fwindows%2Fnanoserver&tag=1809: open //./pipe/docker_engine: The system cannot find the file specified.

## Troubleshooting

1. Is the docker service running? Check with `PS > Get-Service docker`
  Output is

```console
Status	Name	DisplayName
------	----	-----------
Running	Docker	Docker Engine
```

2. Who can access this pipe `docker_engine`? Run `[System.IO.Directory]::GetAccessControl("\\.\pipe\docker_engine") | Format-Table`.
Output is

```powershell
PS C:\Users\user> [System.IO.Directory]::GetAccessControl("\\.\pipe\docker_engine") | Format-Table

Path Owner       Access
---- -----       ------
     domain\user NT AUTHORITY\SYSTEM Allow  FullControl...
```

3. Who can access this pipe `docker_engine_windows`? Run `[System.IO.Directory]::GetAccessControl("\\.\pipe\docker_engine_windows") | Format-Table`.
Output is

```powershell
PS C:\Users\user> [System.IO.Directory]::GetAccessControl("\\.\pipe\docker_engine_windows") | Format-Table

Path Owner                  Access
---- -----                  ------
     BUILTIN\Administrators NT AUTHORITY\SYSTEM Allow  FullControl...
```

## The Fix

Grant the user access only to the Docker Pipe
as in <https://github.com/tfenster/dockeraccesshelper>
and not to the whole machine.

```powershell
# GrantAccessToDockerPipe.ps1
# account that's to be granted access. When on laptop, I use juliusg.
$account="juliusg" 
$npipe = "\\.\pipe\docker_engine"
$dInfo = New-Object "System.IO.DirectoryInfo" -ArgumentList $npipe
$dSec = $dInfo.GetAccessControl()
$fullControl =[System.Security.AccessControl.FileSystemRights]::FullControl
$allow =[System.Security.AccessControl.AccessControlType]::Allow
$rule = New-Object "System.Security.AccessControl.FileSystemAccessRule" -ArgumentList $account,$fullControl,$allow
$dSec.AddAccessRule($rule)
$dInfo.SetAccessControl($dSec)
```

## Testing

Open PowerShell and run that script.

In another PowerShell window, check the following:

1. Is the docker service running? Check with `PS > Get-Service docker`
  Output is

```console
Status	Name	DisplayName
------	----	-----------
Running	Docker	Docker Engine
```

2. If it's not running, restart it with `PS > Get-Service docker | Restart-Service` and not with `dockerd`. 
`dockerd` starts new instance of the Docker Daemon)

## References

- Grant user access to docker pipe <https://github.com/tfenster/dockeraccesshelper> based on <https://www.axians-infoma.de/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/>
- Docker Daemon config: https://docs.docker.com/config/daemon/
    - config.json: `C:\ProgramData\docker\config\daemon.json` on Windows
    - docker daemon directory `C:\ProgramData\docker` on Windows. The Docker daemon persists all data in a single directory. This tracks everything related to Docker, including containers, images, volumes, service definition, and secrets.