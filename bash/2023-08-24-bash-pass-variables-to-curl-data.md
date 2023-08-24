---
title:  'Bash and curl - pass variables to curl'
author: julius
date: '2023-08-24T19:05:00:00+01:00'
tags:
  - 'curl'
  - 'bash'
---

I often call `curl` in a `bash` script.
How do I pass variables that I've set in `bash` as arguments to `curl`'s command line option, say `--data`?

Below is an example without variables.

```bash
curl \
  --header "Content-Type: application/json" \
  --data '{
            "name": "Hello John Doe" 
          }' \
  http://example.com
```

Splitting `John Doe` into two variables, and passing them to `curl` makes the server display the `greeting` as `"Hello $firstname $lastname"`.

```bash
# with variable - doesn't work. Variables are sent in request as is.
# Server displays "Hello $firstname $lastname"
$firstname="John"
$lastname="Doe"

curl \
  --header "Content-Type: application/json" \
  --data '{
            "greeting": "Hello $firstname $lastname",
            "firstname": "$firstname",
            "lastname": "$lastname",
          }' \
  http://example.com
```

I've come across two cases:

- If a variable is part of an already in a double quoted string, enclose in single quotes. 
See the value of key, `greeting`.
- if a variable is to be its own json value, escape the double quotes. See `greeting`. 
See the value of key, `firstname` and `lastname`

```bash
$firstname="John"
$lastname="Doe"

curl \
  --header "Content-Type: application/json" \
  --data '{
            "greeting": "Hello '$firstname' '$lastname'",
            "firstname": '\"$firstname\"',
            "lastname": '\"$lastname\"'
          }' \
  http://example.com
```

Thanks to [ Bernie Michalik's post](https://smartpeopleiknow.com/2021/11/12/if-you-are-writing-a-bash-script-to-call-a-curl-command-and-you-want-to-pass-variable-values-to-it-read-this) for the hints.
