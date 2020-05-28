---
layout: post
title: "Fuzzing Sequoia-PGP"
author: capitol
category: infrastructure
---
![sequoia](/images/sequoia.jpg)

Sequoia is a promising new OpenPGP library that's written in Rust. As Rust has excellent
interoperability with C it also exposes itself as a C library in the [sequoia_openpgp_ffi](https://docs.sequoia-pgp.org/sequoia_openpgp_ffi/index.html)
crate. This would be the way that you would call this library from other programming languages,
as C often acts as the lowest common denominator.

As Sequoia is making progress towards a 1.0 release, I thought that it would be time to help out by
trying to discover bugs in it by fuzzing, a technique where you generate random input to
functions and observe the execution flow in order to detect problems.

When attacking a system it's often useful to determine where the trust boundaries of the system lie,
and the ffi crate is one of those boundaries. Parser code is a large source of bugs, it's very
hard to foresee all different types of invalid input when writing a parser so [pgp_packet_parser_from_bytes](https://docs.sequoia-pgp.org/sequoia_openpgp_ffi/parse/fn.pgp_packet_parser_from_bytes.html)
looked like a prime candidate for fuzzing.

### The Setup

I decided to use the [cargo fuzz framework](https://github.com/rust-fuzz/cargo-fuzz) which turned
out to be a good choice, it's very simple to get started with.

Initial setup:

```bash
cargo install cargo-fuzz
git clone https://gitlab.com/sequoia-pgp/sequoia.git
cd sequioa
cargo fuzz init
```

This creates the setup you need for fuzzing and adds a skeleton file in `fuzz/fuzz_targets/fuzz_target_1.rs`

I implemented the target like this:

```rust
#![no_main]
use libfuzzer_sys::fuzz_target;

extern crate sequoia_openpgp_ffi;

fuzz_target!(|data: &[u8]| {
    if data.len() > 0 {
        let _ = sequoia_openpgp_ffi::parse::pgp_packet_parser_from_bytes(core::option::Option::None, &data[0], data.len());
    }
});
```

The framework generates random data and uses that to call the `fuzz_target` function. I then simply
pass that data onto the `sequoia_openpgp_ffi::parse::pgp_packet_parser_from_bytes` function.

I started to fuzzing like this:

```bash
cargo fuzz run fuzz_target_1 -- -detect_leaks=0
```

The framework also detects memory leaks, and there was some output related to that. But that's not what
I'm looking for today so I disable that with `-- -detect_leaks=0`.

### Findings

It seems like I was the first person to run a fuzzer against that function, so I found the following:

1. An [integer overflow on a shift left](https://gitlab.com/sequoia-pgp/sequoia/-/issues/514)
1. An [attempt to parse invalid UTF-8](https://gitlab.com/sequoia-pgp/sequoia/-/issues/515)
1. An [attempt to read nonexisting data](https://gitlab.com/sequoia-pgp/sequoia/-/issues/516)

The framework is very explicit in telling us what data caused the broken behaviour, so it's trivial
to take that input and build a unit test from it. This helps the maintainers to triage and fix the problems.

It can also try to minimize the test input automatically, so to keep the tests small. But
that functionality seemed to be broken.

### Security Implications

Since this is Rust, these panics don't cause undefined behaviour as they might have done in for example C.
So these findings will at most cause a denial of service due to the process crashing.

### Thanks

Big thanks to the Sequoia team who was very responsive when I reported the issues and especially Neal
on that team.
