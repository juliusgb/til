# Check OS to determine commands to execute

I use different operating systems for work.

- Develop on Windows with Powershell, CMD, and Git Bash.
- Pipeline code runs in the AWS CodeBuild environment,
which we've set to Amazon Linux.
- End result is an EC2 instance, either with Amazon Linux or Windows.

Make automates lots of steps for us.
I use Make on Windows - <https://www.gnu.org/software/emacs/manual/html_node/efaq-w32/GnuWin32.html>

Checking which OS before running commands has been useful,
thanks to <https://stackoverflow.com/a/14777895>.
The code looks like:

```make
ifeq '$(findstring ;,$(PATH))' ';'
    detected_OS := Windows
else
    detected_OS := $(shell uname 2>/dev/null || echo Unknown)
    detected_OS := $(patsubst CYGWIN%,Cygwin,$(detected_OS))
    detected_OS := $(patsubst MSYS%,MSYS,$(detected_OS))
    detected_OS := $(patsubst MINGW%,MSYS,$(detected_OS))
endif

ifeq ($(detected_OS),Windows)
    $env:SOURCE_AMI = '1234'
else
    export SOURCE_AMI="1234")
endif
```

Run the Makefile with `make -f /path/to/Makefile`.

The next step would be to detect which shell I'm using.
