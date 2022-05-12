# Manage packages globally with pipx

Python comes with many tools that help manage versions and dependencies.

- [pyenv](https://github.com/pyenv/pyenv) - manage Python versions
- virtual environments (like [venv](https://docs.python.org/3/library/venv.html#module-venv)) that help isolate dependencies on a per project basis

What about "global" packages like code linters (flake8, pylint), code formatter (black), or virtualennwrapper (virtual environment manager)?

That question came up when I wanted to use [cfn-lint](https://github.com/aws-cloudformation/cfn-lint) for linting Cloudformation.
Regardless of what project I'm working on, I'd like to have `cfn-lint` ready.

When [installing pipx](https://pypa.github.io/pipx/installation/),
I took the option of changing where `pipx` installs packages through the
two environment variables `PIPX_BIN_DIR` and `PIPX_HOME`:

- Set `PIPX_BIN_DIR` to `C:\opt\Python\pipx\pipx-apps-bins`
- Set `PIPX_HOME` to `C:\opt\Python\pipx\pipx-venv`

Install `pipx` with `PS> python -m pip install --user pipx` and then `PS> python -m pipx ensurepath`.

The output should be: `Success! Added C:\opt\Python\pipx\pipx-apps-bins to the PATH environment variable.`

Now, when I install a package globally, say `PS> python -m pipx install cfn-lint`,
I know where to find `cfn-lint`.
I find it easier to remeber `C:\opt` rather than `~/.local/*` (sometimes Windows hides folders prefixed with a dot)
