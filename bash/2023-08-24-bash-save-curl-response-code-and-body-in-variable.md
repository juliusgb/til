---
title:  'Bash - save curl response code and body in variables'
author: julius
date: '2023-08-24T19:05:00:00+01:00'
tags:
  - 'curl'
  - 'bash'
---

Save curl httpcode and response in variables for further processing.
Found via https://unix.stackexchange.com/a/572434

```bash
#!/bin/bash

URL="https://example.com"

response=$(curl --silent --write-out "%{http_code}" $URL)

http_code=$(tail -n1 <<< "$response")  # get the last line
content=$(sed '$ d' <<< "$response")   # get all but the last line which contains the status code

echo "$http_code"
echo "$content"

# do something with the http_code, such as
if [[ "http_code" -ne 201 ]]; then
    echo "Create failed."
    exit 1
fi
```

Links to the command line options:

- [`-s, --silent`](https://curl.se/docs/manpage.html#-s)
- [`-w, --write-out`](https://curl.se/docs/manpage.html#-w)
