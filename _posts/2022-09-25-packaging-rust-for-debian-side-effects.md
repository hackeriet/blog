---
layout: post
title: "Packaging Rust for Debian - Side Effects"
author: capitol
category: infrastructure
---
![rusted-metal-disk-top](/images/fuzzy-growth-on-rusted-metal-disk-top.jpg)

When packaging a rust crate for Debian, bug fixing is a part of the
process. There is of course multiple sources of bugs, but a common
one is that the crate doesn't build on all architectures.

The builds in Debian happens on a lot of different architectures, and
normally you only have an x86_64 machine to test on before pushing a new
version of a package.

The full list of different architectures is:

* amd64
* arm64
* armel
* armhf
* i386
* mips64el
* mipsel
* ppc64el
* s390x
* alpha
* arc
* hppa
* hurd-i386
* ia64
* kfreebsd-amd64
* kfreebsd-i386
* m68k
* powerpc
* ppc64
* riscv64
* sh4
* sparc64
* x32

But it's not mandatory that the build succeeds on all of them, only the top 9
have `Supported` status and the rest are more of a nice-to-have.

As an example I got this failure on riscv64 recently:

```
failures:

---- src/xy.rs - xy::XY<bool>::both (line 502) stdout ----
malloc_consolidate(): unaligned fastbin chunk detected
Couldn't compile the test.

failures:
    src/xy.rs - xy::XY<bool>::both (line 502)
```

What happens then is that I will investigate the error, try to produce
a patch for it, and send that as a pull request to the upstream sources.

If all is well and good the patch is accepted and included in the next
release of that package. But releases are often infrequent and to stop
the packaging effort of everything that depends on the package is too
time consuming.

Therefore we add the fix as a patch to the Debian package directly. This
keeps the original part of the debian version number and bumps the
debian part, version `0.3.4-1` becomes `0.3.4-2`.

Normally this is an uncomplicated process, a new file is dropped into
the `patches` directory, it's added to the end of the `series` file
in that directory, and then it's shipped off to the build farm.

But there is a not very well documented gotcha in this process,
when the debian rust tooling produces an updated package it reuses the
original `.orig.tar.gz` file from the `-1` package. This means that
all changes that would alter the content of that file are ignored.

For example if you add a line like this to `debcargo.conf` in an `-2` update:
```toml
excludes = ["examples/**", "benches/**"]
```

That will not have any effect, until a new upstream release is made and that
gets packaged.

To summarize, be careful when adding new configuration that alters the original
source code package when you are fixing bugs.

![rusted-metal-disk-bottom](/images/fuzzy-growth-on-rusted-metal-disk-bottom.jpg)
