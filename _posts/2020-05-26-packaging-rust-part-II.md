---
layout: post
title: "Packaging Rust for Debian - part II"
author: capitol
category: infrastructure
---
![rusty-steel](/images/stainless_steel_iron_metal_rusty_weathered.jpeg)

Let's do another dive into packaging Rust for Debian with a slightly more complicated example.

One great tool in the packager's toolbox is [cargo-debstatus](https://crates.io/crates/cargo-debstatus).
By running it in the root of your crate you will get a list of your dependencies, together
with information regarding its packaging status in Debian.

For `ripasso` a part of the tree looks like below at the time of writing. `ripasso` depends on `gpgme`
which depends on a number of other Rust libraries (and a number of native ones which aren't shown here).
```
├── gpgme v0.9.2
│   ├── bitflags v1.2.1 (in debian)
│   ├── conv v0.3.3
│   │   └── custom_derive v0.1.7
│   ├── cstr-argument v0.1.1 (in debian)
│   ├── gpg-error v0.5.1 (in debian)
│   ├── gpgme-sys v0.9.1
│   │   ├── libc v0.2.66 (in debian)
│   │   └── libgpg-error-sys v0.5.1 (in debian)
│   ├── libc v0.2.66 (in debian)
│   ├── once_cell v1.3.1 (in debian)
│   ├── smallvec v1.1.0 (in debian)
│   └── static_assertions v1.1.0
```

One of the dependencies is [static_assertions](https://crates.io/crates/static_assertions) which
is actually already packaged in Debian, but version 0.3.3 and we need 1.1.0. Let's investigate how
to fix this one.

In order to verify that we won't break any other package by upgrading the existing Debian package
to 1.1.0 we run [list-rdeps.sh](https://salsa.debian.org/rust-team/debcargo-conf/-/blob/master/dev/list-rdeps.sh).

```
$ ./dev/list-rdeps.sh static-assertions
APT cache is a bit old, update? [Y/n] n
Versions of rust-static-assertions in unstable:
  librust-static-assertions-dev                    0.3.3-2

Versions of rdeps of rust-static-assertions in unstable, that also exist in testing:
  librust-lexical-core-dev                         0.4.3-1+b1       depends on     librust-static-assertions-0.3+default-dev (>= 0.3.3-~~),
```

Here we see that the [lexical-core](https://crates.io/crates/lexical-core) package depends on static-assertions
and a quick `git clone` and compilation confirms that it doesn't compile with version 1.1.0.

We can take this one step further

```
$ ./dev/list-rdeps.sh lexical-core
APT cache is a bit old, update? [Y/n] n
Versions of rust-lexical-core in unstable:
  librust-lexical-core+correct-dev                 0.4.3-1+b1
  librust-lexical-core+default-dev                 0.4.3-1+b1
  librust-lexical-core-dev                         0.4.3-1+b1
  librust-lexical-core+dtoa-dev                    0.4.3-1+b1
  librust-lexical-core+grisu3-dev                  0.4.3-1+b1
  librust-lexical-core+ryu-dev                     0.4.3-1+b1
  librust-lexical-core+stackvector-dev             0.4.3-1+b1

Versions of rdeps of rust-lexical-core in unstable, that also exist in testing:
  librust-lexical-core+correct-dev                 0.4.3-1+b1       depends on     librust-lexical-core+table-dev (= 0.4.3-1+b1),
  librust-lexical-core+default-dev                 0.4.3-1+b1       depends on     librust-lexical-core+correct-dev (= 0.4.3-1+b1), librust-lexical-core+std-dev (= 0.4.3-1+b1),
  librust-nom+lexical-core-dev                     5.0.1-4          depends on     librust-lexical-core-0.4+default-dev,
  librust-nom+lexical-dev                          5.0.1-4          depends on     librust-lexical-core-0.4+default-dev,
```

And we see that nom depends on lexical-core.

```
$ ./dev/list-rdeps.sh nom
APT cache is a bit old, update? [Y/n] n
Versions of rust-nom in unstable:
  librust-nom+default-dev                          5.0.1-4
  librust-nom-dev                                  5.0.1-4
  librust-nom+lazy-static-dev                      5.0.1-4
  librust-nom+lexical-core-dev                     5.0.1-4
  librust-nom+lexical-dev                          5.0.1-4
  librust-nom+regex-dev                            5.0.1-4
  librust-nom+regexp-dev                           5.0.1-4
  librust-nom+regexp-macros-dev                    5.0.1-4
  librust-nom+std-dev                              5.0.1-4

Versions of rdeps of rust-nom in unstable, that also exist in testing:
  librust-cexpr-dev                                0.3.3-1+b1       depends on     librust-nom-4+default-dev, librust-nom-4+verbose-errors-dev,
  librust-dhcp4r-dev                               0.2.0-1          depends on     librust-nom-5+default-dev (>= 5.0.1-~~),
  librust-iso8601-dev                              0.3.0-1          depends on     librust-nom-4+default-dev,
  librust-nom-4+lazy-static-dev                    4.2.3-3          depends on     librust-nom-4-dev (= 4.2.3-3),
  librust-nom-4+regex-dev                          4.2.3-3          depends on     librust-nom-4-dev (= 4.2.3-3),
  librust-nom-4+regexp-macros-dev                  4.2.3-3          depends on     librust-nom-4-dev (= 4.2.3-3), librust-nom-4+regexp-dev (= 4.2.3-3),
  librust-nom-4+std-dev                            4.2.3-3          depends on     librust-nom-4-dev (= 4.2.3-3), librust-nom-4+alloc-dev (= 4.2.3-3),
  librust-nom+default-dev                          5.0.1-4          depends on     librust-nom+std-dev (= 5.0.1-4), librust-nom+lexical-dev (= 5.0.1-4),
  librust-nom+regexp-macros-dev                    5.0.1-4          depends on     librust-nom+regexp-dev (= 5.0.1-4),
  librust-nom+std-dev                              5.0.1-4          depends on     librust-nom+alloc-dev (= 5.0.1-4),
  librust-pktparse-dev                             0.4.0-1          depends on     librust-nom-4+default-dev (>= 4.2-~~),
  librust-rusticata-macros-dev                     2.0.4-1          depends on     librust-nom-5+default-dev,
  librust-tls-parser-dev                           0.9.2-3          depends on     librust-nom-5+default-dev,
  librust-weedle-dev                               0.10.0-3         depends on     librust-nom-4+default-dev,
```

And a lot of things depends on nom.

So in order to package `static_assertions` so that we can package `gpgme` we can choose one of three
different strategies:

1. Package both versions of `static_assertions`
1. Upgrade `lexical-core`, `nom` and everything `nom` depends on to newer versions
1. Patch version 0.4.3 of `lexical-core` to use a newer version of `static_assertions`

### Packaging both versions of `static_assertions`

This is a working strategy, but packaging both means that we need to create a new package for
version 0.3 of `static_assertions`. New packages in Debian go through the new queue, where a member
of the ftp masters team needs to manually verify so that it doesn't contain any non-free software.

Therefore we will not choose this strategy.

### Upgrading `lexical-core`, `nom` and everything `nom` depends on to newer versions

There exists a new version of `lexical-core` that depend on `static_assertions` 1, but the newer
version of `nom` is a beta version of `nom` 6, and upgrading to that version would mean that we would
need to patch all the incompatibilities in the applications that use nom.

A lot of non-trivial work, specially as we in that case would like to upstream the patches so that
the maintenance burden doesn't grow too much.

### Patching version 0.4.3 of `lexical-core` to use a newer version of `static_assertions`

It turns out that there is an upgrade [commit](https://github.com/Alexhuszagh/rust-lexical/commit/1e2b1ab6561903e44b5fdaef923e1a1c1f79148d)
in `lexical-core` that applies cleanly to version 0.4.3. This is what we will use.

So we take that commit as a patch and place it into the [patches directory](https://salsa.debian.org/rust-team/debcargo-conf/-/tree/master/src/lexical-core/debian/patches)
together with a [series file](https://salsa.debian.org/rust-team/debcargo-conf/-/blob/master/src/lexical-core/debian/patches/series)
that just lists what order to patches should be applied in.

That enables us to upgrade the `static_assertions` package to version 1.1.0 without breaking
any other package.
