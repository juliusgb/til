# 'Packer - whitespace in env vars

When setting [environment variables Packer](https://developer.hashicorp.com/packer/docs/provisioners/powershell#environment_vars), make sure there's no space in between.
If there's a whitespace, Packer passes on that whitespace within the value of the env var.

Using `#` as a marker, we can see what value our env var will look like:

```hcl
  provisioner "powershell" {
    environment_vars = [
      "ENV_VAR_WITHOUT_SPACE=${var.tmp_folder}",
      "ENV_VAR_WITH_SPACE = ${var.tmp_folder}"
    ]
    inline = [
      "$ErrorActionPreference='Stop'",
      "Write-Host \"EnvVarWithoutSpace:#$Env:ENV_VAR_WITHOUT_SPACE\"",
      "Write-Host \"EnvVarWithSpace:#$Env:ENV_VAR_WITH_SPACE\""
    ]
  }
```

output is

```console
. . .
EnvVarWithoutSpace:#C:\tmp\packer-env-var
EnvVarWithSpace:# C:\tmp\packer-env-var
```

Another way to deal with this is to call the `Trim` method before processing those env vars:

```powershell
# check if they exist
$expectedEnvVars = @(
    $env:ENV_VAR_WITHOUT_SPACE, 
    $env:ENV_VAR_WITH_SPACE
)
foreach ($envVar in $expectedEnvVars) {
    if ([string]::IsNullOrWhiteSpace($envVar)) {
        throw "$envVar is null or is empty" 
    }
}
$MyVar1 = ($env:ENV_VAR_WITHOUT_SPACE).Trim()
$MyVar2 = ($env:ENV_VAR_WITH_SPACE).Trim()

# do something with $MyVar1 and $MyVar2
```
