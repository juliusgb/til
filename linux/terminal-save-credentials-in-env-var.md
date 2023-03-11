# Shell Terminal - save credentials in env var

Sometimes I need to save credentials in an environment variable that I can later use in a script.

```sh
# prompt for user for username.
read -s -p "Enter username: " SECRET_USER
# User types the username

# export as env var
export SECRET_USER

# prompt user for password
read -s -p "Enter password: " SECRET_PASSWORD
# User types the username

# export as env var
export SECRET_PASSWORD

# use in script
curl --user $SECRET_USER:$SECRET_PASSWORD $etc
```

Helpful link: https://stackoverflow.com/a/3980904