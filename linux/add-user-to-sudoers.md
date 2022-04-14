# Add user to sudoers

How do I enable a user, say `userx`, to run commands prefixed with `sudo` ?
After running `sudo` as `userx`, I also don't want to enter a password.

In CentOS, add the user to the `wheel` group.

```
echo "userx ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/userx
```

Source: https://linuxize.com/post/how-to-add-user-to-sudoers-in-centos/