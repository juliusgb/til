# Powershell - set env vars in different scopes

While trying to understand [this line](https://github.com/actions/runner-images/blob/035dc70f77d4353de9fbacd66be4f0797d4d9f62/images/win/scripts/ImageHelpers/PathHelpers.ps1#L63) from the utilities that GitHub uses for their GitHub Runners, it occurred to me that environment variables are defined in different contexts that affects how long they last.

Thanks to [shellgeek](https://shellgeek.com/set-environment-variable-using-powershell/), there's a way to set the env vars in PowerShell accordingly:

- Env vars for the duration of the powershell session and process:
  `$Env:GoHugo = "C:\Hugo\bin\"`

- Env var for the User account (System Settings -> Advanced -> Environment Variables -> User): 
  `[System.Environment]::SetEnvironmentVariable('GoHugo','C:\Hugo\bin',[System.EnvironmentVariableTarget]::User)`

- Env var for the machine in (System Settings -> Advanced -> Environment Variables -> System): 
  `[System.Environment]::SetEnvironmentVariable('GoHugo','C:\Hugo\bin',[System.EnvironmentVariableTarget]::Machine)`
