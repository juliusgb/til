---
title:  'Bash - declare and loop over array'
author: julius
date: '2023-08-24T19:05:00:00+01:00'
tags:
  - 'bash'
---

How to declare an array and loop over it?

```bash
# declare array: https://www.gnu.org/software/bash/manual/html_node/Arrays.html
declare -a myArray=("Linux Mint" "Fedora" "Red Hat Linux" "Ubuntu" "Debian" )

# loop through it: https://www.freecodecamp.org/news/bash-array-how-to-declare-an-array-of-strings-in-a-bash-script/
for str in ${myArray[@]}; do
  echo $str
done
```

Sources:

- declare array: https://www.gnu.org/software/bash/manual/html_node/Arrays.html
- loop through an array: https://www.freecodecamp.org/news/bash-array-how-to-declare-an-array-of-strings-in-a-bash-script/
