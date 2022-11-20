---
layout: post
title: "Release of Ripasso version 0.6.0"
author: capitol
category: infrastructure
---
![ripasso-cursive](/images/ripasso-cursive-0.4.0.png)

Time passes as water under a bridge, it is yet again to release a version of ripasso. We present version 0.6.0.

## New Features

#### Choosable OpenPGP backend

We have implemented support for configuration files. You can now switch between different password
directories from the menu.

#### Experimental new OpenPGP backend based on Sequoia

The Sequoia project is a implementation of OpenPGP written in Rust. Since the GPG project
have suffered from multiple security problems due to memory corruption lately it's time
to start exploring alternative implementations.

The Sequoia backend can be enabled per store by adding `pgp_implementation = 'sequoia'` in
your config file.

The sequoia implementation doesn't support reading the gpg keyring on the system, so if that
one in chosen all public pgp keys must be imported imported into ripasso. But it can talk
to the gpg-agent, so it doesn't require that the private key is imported into sequoia.

#### Support for TOTP codes

Ripasso now supports otpauth urls, if there is a url on the format `otpauth://` then a MFA token
can be copied with `ctrl-b`.

#### Support the wayland copy buffer

If running ripasso in a wayland environment, we now support the wayland copy buffer.

#### Comments in .gpg-id file

The Pass project have added support for comments in the `.gpg-id` file, comments start
with a `#` character.

#### Download OpenPGP certs from keys.openpgp.org

Added support for downloading pgp keys from keys.openpgp.org.

## Bugs Fixed

#### Compression in gpg

Disabled compression when encrypting secrets with gpg. Compressing before encrypting can sometimes
lead to a compression oracle vulnerability. These kind of vulnerabilities typically require that
an attacker can automate the creation of secrets in some way, so we don't think it's applicable here.

#### Copy behaviour between <enter> and <ctrl-y>

Enter now copies the first line of the secret, and ctrl-y copies the whole secret.

## Credits

 * Joakim Lundborg - Developer
 * Alexander Kjäll - Developer

Also a big thanks to everyone who contributed with bug reports and patches.
