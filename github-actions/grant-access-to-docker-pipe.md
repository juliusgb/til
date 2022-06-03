# Grant user access to Docker Pipe

Goals:

- able to run `docker` commands without getting `Administrator` privileges.
- Run `docker` commands in a GitHub Action that's running on Self-hosted runners.

When running `docker build` as part of a step in a GitHub action, we're greeted with the error

> error during connect: In the default daemon configuration on Windows, the docker client must be run with elevated privileges to connect.: Post http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/images/create?fromImage=mcr.microsoft.com%2Fwindows%2Fnanoserver&tag=1809: open //./pipe/docker_engine: The system cannot find the file specified.

## The Fix

Grant the user under which the GitHub Action runs access only to the Docker Pipe
as in <https://www.axians-infoma.de/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/>
and not to the whole machine.

Doing that as part of preparing the runner means I don't need to think about it anymore.

## The Workflow

The github action checks out the repository to access a couple of files:

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
    - name: Display User
      run: |
        whoami
    - name: Run docker build
      run: docker build --file "$env:GITHUB_WORKSPACE\Dockerfile-Windows" --tag localhost:5000/mydockerapp:0.0.1
```

## References

- Grant user access to docker pipe <https://www.axians-infoma.de/techblog/allow-access-to-the-docker-engine-without-admin-rights-on-windows/>
