---
layout: post
title: "Solution to Bornhack 2020 CTF challenge nc3"
author: capitol
category: ctf
---

![tent_island](/images/tent_island.jpg)

##### Name:
nc3

##### Category:
crypto

##### Points:
25

#### Writeup

This was the starting challenge in the crypto category, just a couple of different encodings.

Challenge text:

```
base64:cmV2ZXJzZTo5UldZaTkxYno5RmR1TlhZMzlGZGhoR2Q3WkVWRGhrUTo0NmVzYWI=
```

We solved it with bash:

```bash
#!/bin/bash

cat nc3 | awk '{ print(substr($0, 8, length($0)))}'|base64 -d |\
    awk '{ print(substr($0, 9, length($0)))}'|rev|\
    awk '{ print(substr($0, 8, length($0)))}'|base64 -d
echo
```

Flag was `BHCTF{that_wasnt_so_bad}`.