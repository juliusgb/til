# Packer - setup local dev environment on Windows using Packer

Aim: automate setting up my local dev environment on Windows using HashiCorp's [Packer](https://www.packer.io)

My laptop frequently breaks down. After having it repaired, I have to reinstall most programs I need for development. Why not have an automated way to install everything.

I also want to be able to open a shell, execute commands to run programs without having first to worry about installing and configuring the tools/SDKs/Runtimes.

This TIL's description ended up the [repository's readme](https://github.com/juliusgb/dev-laptop-setup).

Prerequisites:

- `Winrm` is to Windows what `ssh` is to Linux. So `Winrm` has been setup and tested.
Just as Packer uses `ssh` to remotely manage Linux machines, so does [Packer use `winrm` to remotely manage Windows machines](https://www.packer.io/docs/communicators/winrm).
To set up `winrm`, follow the TIL, <https://github.com/juliusgb/til/blob/main/winrm/setup-winrm.md>
- You've setup and tested Packer with WinRM. See TIL, <https://github.com/juliusgb/til/blob/main/packer/integrate-with-winrm.md>, on how to do that.

At the end of this TIL, after we run the Packer template, we'll have these tools installed and configured: <https://github.com/juliusgb/dev-laptop-windows/blob/main/images/win/Windows2022ish-Readme.md>


That's a subset of the tools that Microsoft installs on [its virtual machines](https://github.com/actions/virtual-environments) on which GitHub Actions run.
I wanted something similar, i.e., open a shell, execute commands to run programs without having first to worry about installing and configuring the tools/SDKs/Runtimes. That worry comes later.

I'm using release [`Windows Server 2022 (20220710 update)`](https://github.com/actions/virtual-environments/releases/tag/win22%2F20220710.1).
But I've removed server-specific settings and tools, such as [`sbt`](https://www.scala-sbt.org/), that I'm not likely to use.
If I need the tool, I'll add the relevant `Install-*` script and corresponding [`Pester`](https://pester.dev/) tests to the repo.

## Getting started

We need the following:

- An internet connection that's not so firewalled.
If network access is locked down via firewalls, remember to add the URLs in the `Install-*` scripts to the firewall's `allowlist`.
Also keep in mind that some of these URLs get redirected. Then, add the redirects to the `allowlist`.
To see all redirects for a URL, use the [`UrlHelpers`](https://github.com/juliusgb/utils/blob/main/powershell/CustomHelperUtils/UrlHelpers.ps1) script.

- Able to open and run PowerShell as Administrator. In it, elevate the [PowerShell session's execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) with `Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope Process`
- The Packer executable. It doesn't have to be on the PATH. Download it from <https://www.packer.io/downloads>.

- `Winrm` has been setup and tested. `Winrm` is to Windows what `ssh` is to Linux.
Just as Packer uses `ssh` to remotely manage Linux machines, so [Packer uses `winrm` to remotely manage Windows machines](https://www.packer.io/docs/communicators/winrm).
To set up `winrm`, follow the post, <https://til.juliusgamanyi.com/posts/setup-winrm>


**WARNING**
Running scripts with Administrator privileges triggers an alarm at the back of my mind.
So read them, test them. And buyer beware!

## Testing

To change these scripts, we need portable versions of some tools, like Packer, 7zip, vscode, git.

There's no one script (yet) to bootstrap everything.
To test manually, do the following:

1. Setup `winrm`. In PowerShell, run `.\winrm\SetupWinRmForPacker.ps1`
2. Validate the packer template file with `C:\path\to\packer.exe validate packer\template.pkr.hcl`
3. Packer build the template file with `C:\path\to\packer.exe build packer\template.pkr.hcl`.
:zap: Read the section "more on step 3" :zap:
4. Cleanup what was added during the `winrm` setup. In PowerShell, run `.\winrm\CleanupWinrmSetupForPacker.ps1`

### More on step 3

Changes to the packer template file mean re-running `packer build`.
That's a once-for-all operation: there are no intermediate caches for previous steps to use again.

- The `Install-*` scripts use [`Chocolatey`](https://chocolatey.org/) to install the dev tools. And `Chocolatey` knows whether a tool was installed.
If I re-run the `choco install` step without the `--force` option, `Chocolatey` doesn't reinstall it.
Yay, for [Idempotency](https://en.wikipedia.org/wiki/Idempotence)!! :sparkles:

- What's not idempotent are the directories and environment variables that were created during installation.
	- That's ideal when starting from a fresh, clean machine. But have to take more care when running on my one dev machine (laptop) - no immutability.
    - One way I re-test is to manually delete them before re-running `packer build`.
	- Another way is that when testing 2 lines, and the 2nd fails, I comment out line 1 and re-run `packer build`, which executes only line 2. That leaves the directories and env vars from line 1 untouched.

## Customisations

Besides keeping a subset of tooling, I've customised some to match my preferences.

- I prefer the tooling to be installed in `C:\opt` instead of `C:\`.
This meant changing related files, such as the tests and the scripts that generate the reports for the installed software.

- I've commented out some files instead of deleting them.
These are for tooling that I'm _likely_ (or would like) to use.
Their corresponding tests are `skipped`.

- For tooling installed with Chocolatey, I prefer the [`<package>.portable` instead of the `<package>.install` versions](https://docs.chocolatey.org/en-us/faqs#what-distinction-does-chocolatey-make-between-an-installable-and-a-portable-application)
that don't require Admin rights to install nor integration with Windows file explorer.
Some Chocolatey packages allow you to change where they're installed. For these ones, I install them in `C:\opt`
I use the `default` or `<package>.install` version for those that need Admin rights (Git, 7zip) or that need integration with Windows File Explorer or whose installer is to cumbersome to change (CMake, AWS CLIs).

## Excluded Tooling

- PyPy because it failed to create the `Scripts` directory for version 3.8 and 3.9.
I didn't have bandwidth to investigate and fix. Besides, Python's installation worked.
Excluding PyPy also meant removing the corresponding tests in `Toolset.Tests.ps1` and
from the the `toolset-2022.json` (the tests use it as input for some of its checks).

- Docker. The installer installs it via OneGet.
This installs only Windows Containers.
On the other hand, Docker Desktop allows us to choose which containers to build and run - we can choose Windows or Linux containers.
It's better to install Docker Desktop via Chocolatey.

Because of excluded tooling, customized installers, I've added the `-ish` suffix to the file listing the installed tooling, i.e., to `Windows2022ish`.


## References

- Setting up WinRM and manually testing it - https://til.juliusgamanyi.com/posts/setup-winrm
- Setup Packer with WinRM - https://til.juliusgamanyi.com/posts/setup-local-dev-with-packer
- Use Packer to install dev tooling - https://til.juliusgamanyi.com/posts/use-packer-to-install-tools-on-dev-machine
- Virtual machines for GitHub Actions - https://github.com/actions/virtual-environments
- Best script was `Configure-Toolset.ps1`. It's idempotent.
It checked the directories and registry keys, deleting them, if they already exist.
That made testing (i.e., re-running it) easy.
