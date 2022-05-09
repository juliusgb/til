# GNU Make on Windows

Why use `make` in the first place?

- In our team, some use Windows while others use Mac and steps to run are in a Makefile.
- The CI/CD environment supports both Windows and Linux-based steps.
- I'd like such one file that I can run locally and on CI, on Windows and on Mac.

Which version of `make` should I use for Windows?

- I chose <https://github.com/mbuilov/gnumake-windows/blob/master>
- The "official" version 3.81 at <http://gnuwin32.sourceforge.net/packages/make.htm> was last updates in 2006 and has a known bug <https://make-w32.gnu.narkive.com/rSJDAHlg/process-begin-createprocess-null-failed>

Which shell to use?

- On Windows, use PowerShell Core because it's cross platform - works on Linx and Mac too. Windows PowerShell comes with Windows but is not cross platform. Differences are listed here in the [docs](https://docs.microsoft.com/en-us/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.2)
- After downloading the [Zip](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2#zip), I extract it to `C:\opt\PowerShell\7\` and add it to the `PATH`

Things to note:

- Tabs vs Spaces. [Stackoverflow](https://stackoverflow.com/a/28720186) has the explanation.
  - Without fixing this, when `make` runs, it reports error, `*** missing separator.`
  - Enable this in VSCode: View -> Render Whitespace
  - Because of this, I add an `.editorconfig`(see [utils/.editorconfig](https://github.com/juliusgb/utils/blob/main/.editorconfig)) that enforces tabs for Makefiles and manually add spaces when needed.
  - As an example of this mix, take a look at <https://github.com/juliusgb/utils/blob/main/make/Makefile>
  - Because of this, I run Make files in PowerShell or CMD and not in Git Bash.
- When in a Makefile, use the inbuild variable `$(MAKE)` to run other `make` commands/files.

## Next steps

The next step would be to detect which shell I'm using.

## Useful links

- The manual: <https://www.gnu.org/software/make/manual/make.html>

- Collection of useful snippets and corresponding links: <https://vinta.ws/code/use-makefile-as-the-task-runner-for-arbitrary-projects.html>
