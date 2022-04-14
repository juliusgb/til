# Show details of Operating System

I wanted to know what Operating System I'm using in order to use OS-specific utilities like [adding users to sudoers](add-user-to-sudoers.md)

Execute:
```
$ cat /etc/os-release
```

Output:
```
-------OS Information--------
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
------------------------------
```
