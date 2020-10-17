---
layout: post
title: "Release of Ripasso version 0.5.0"
author: capitol
category: infrastructure
---
![ripasso-cursive](/images/ripasso-cursive-0.4.0.png)

After nine long months of development effort, we are proud to present
ripasso version 0.5.0.

## New Features

#### Support multiple password stores

We have implemented support for configuration files. You can now switch between different password
directories from the menu.

![directory-menu](/images/ripasso-directory-menu.png)

#### Fuzzing of our dependencies

We did a small project where we went over our dependencies with a fuzzer, bugs found:

 * [panic on unwrap of empty string](https://github.com/jeaye/ncurses-rs/issues/196)
 * [panic of unwrap() on CString creation](https://github.com/ihalila/pancurses/issues/77)
 * [byte index 1 is not a char boundary](https://github.com/gyscos/cursive/issues/489) gyscos/cursive
 * [decoding invalid utf8](https://github.com/hjson/hjson-rust/issues/19)
 * [subtract with overflow](https://github.com/hjson/hjson-rust/issues/20)
 * [removal index (is 0) should be < len (is 0)](https://github.com/hjson/hjson-rust/issues/21)
 * [called Result::unwrap() on an Err](https://github.com/hjson/hjson-rust/issues/22)
 * [called Option::unwrap() on a None](https://github.com/zonyitoo/rust-ini/issues/75)
 * [Unrecognized literal: 6E--5458](https://github.com/dtolnay/syn/issues/897)
 * [shift left with overflow](https://gitlab.com/sequoia-pgp/sequoia/-/issues/514)
 * [byte index 11 is not a char boundary](https://gitlab.com/sequoia-pgp/sequoia/-/issues/515)
 * [read empty buffer](https://gitlab.com/sequoia-pgp/sequoia/-/issues/516)
 * [read empty buffer](https://gitlab.com/sequoia-pgp/sequoia/-/issues/517)

Some of them have been closed, some are in optional dependencies that we now have excluded
and some are in a package that we want to start using in the future.

#### Password History View

If you press ctrl-H on a password entry, it will bring up the git history of that file.

![password-history](/images/ripasso-password-history.png)

#### Copy password file name

Copy the file name with ctrl-U, this can be useful if you have your username as the filename.

## Bugs Fixed

#### Passwords in initial commit causes error

If the initial git commit contained files, that caused errors as ripasso didn't consider that
snapshot correctly.

#### Not assume that git branch should be named master

A hardcoding of the branch name was removed.

## Credits

 * Joakim Lundborg - Developer
 * Alexander Kjäll - Developer
 * Silje Enge Kristensen - Norwegian bokmål translation
 * Camille Victor Prunier - French translation
 * David Plassmann - German translation

Also a big thanks to everyone who contributed with bug reports and patches.
