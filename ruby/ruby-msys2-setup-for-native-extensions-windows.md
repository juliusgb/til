# Use MSYS2 to install Ruby dev packages to build native extensions on Windows

Aim: Use [MSYS2](https://www.msys2.org/) to install the [development packages](https://github.com/oneclick/rubyinstaller2/blob/master/lib/ruby_installer/build/components/03_dev_tools.rb) Ruby needs to build native extensions on Windows.

Why? because I found out that some Ruby gems must be build as native extensions. Without this setup, installing such gems never works.

Pre-requisites: [Ruby](https://www.ruby-lang.org/en/documentation/) is installed.

## Background and Problem

After setting up my laptop according to <https://github.com/juliusgb/dev-laptop-setup>, I cloned my [Jekyll](https://jekyllrb.com/) site that's based on [Simply Jekyll](https://github.com/raghudotcc/simply-jekyll) site, ran `gem install bundler jekyll && bundle install`, and was greeted with the error:

```console
In Gemfile:
  jekyll-feed was resolved to 0.16.0, which depends on
    jekyll was resolved to 4.0.1, which depends on
      em-websocket was resolved to 0.5.3, which depends on
        http_parser.rb
mingw32-make: *** [makefile:5: prep] Error 5
```

Manually installing the `http_parser.rb` gem with command `gem install http_parser.rb --version 0.6.0` returned

```console
Temporarily enhancing PATH for MSYS/MINGW...
Building native extensions. This could take a while...
ERROR:  Error installing http_parser.rb:
        ERROR: Failed to build gem native extension.
```

From [stackoverflow](https://stackoverflow.com/a/50929914), I found out that 
>The error indicates that Ruby needs to compile native (non-Ruby) code in order to install and use the http_parser gem. To compile native code on Windows you need to install Ruby with the DevKit package. ---2018, https://stackoverflow.com/a/50929914

Why the DevKit? Because the [RubyInstaller DevKit](https://rubyinstaller.org/add-ons/devkit.html) contains development packages required to compile with Ruby C-extensions.

But according to the [DevKit documentation]((https://rubyinstaller.org/add-ons/devkit.html)), 
> Stating with RubyInstaller-2.4 we’re no longer using our own DevKit compilation, but make use of MSYS2 for both building Ruby itself as well as building Ruby gems with C-extensions.

So I need do the following:

- Make sure that [MSYS2](https://www.msys2.org/) is installed.
- Install the [development packages](https://github.com/oneclick/rubyinstaller2/blob/master/lib/ruby_installer/build/components/03_dev_tools.rb) Ruby needs to needs build with C-extensions. Since [`ridk`](https://github.com/oneclick/rubyinstaller2/wiki/The-ridk-tool) was installed during Ruby's installation, running `ridk install 3` installs the relevant packages.


## Solution

Use [MSYS2](https://www.msys2.org/) to install the [development packages](https://github.com/oneclick/rubyinstaller2/blob/master/lib/ruby_installer/build/components/03_dev_tools.rb) Ruby needs to build native extensions on Windows.

### Using the Dev-laptop Setup repo

I setup my laptop using <https://github.com/juliusgb/dev-laptop-setup>.
I added MSYS2 to it in a [pull request](https://github.com/juliusgb/dev-laptop-setup/pull/10/files), and re-ran `packer build`.

### Using Chocolatey

Another option is to use Chocolatey - <https://community.chocolatey.org/packages/msys2>.

- Open PowerShell as Administrator
- If you don't have Ruby installed, run `choco install ruby`
- Run `choco install msys2 --params "/NoUpdate /InstallDir:C:\your\install\path"`
- Refresh the shell's environment variables with `refreshenv`
- Use the RubyInstaller2 `ridk` to install only the development packages that Ruby needs build with C-extensions.
Run `ridk install 3`
- Check that MSYS was correctly installed by running `ridk version`. The output should show `msys2` and the path to it.

  ```console
    C:\Users\juliu>ridk version
    ---
    ruby:
      path: C:/opt/tools/Ruby/3.0.4/x64
      version: 3.0.4
      platform: x64-mingw32
      cc: gcc.exe (Rev10, Built by MSYS2 project) 11.2.0
    ruby_installer:
      package_version: 3.0.4-1
      git_commit: 17502f9
    msys2:
      path: C:\opt\msys2
    cc: gcc (Rev3, Built by MSYS2 project) 12.1.0
    sh: GNU bash, Version 5.1.16(1)-release (x86_64-pc-msys)
    os: Microsoft Windows [Version 10.0.19044.1826]
  ```
## Installing gems that are built as native extensions

Running `gem install http_parser.rb --version 0.6.0` works without errors