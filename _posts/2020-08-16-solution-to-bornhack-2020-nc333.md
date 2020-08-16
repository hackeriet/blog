---
layout: post
title: "Solution to Bornhack 2020 CTF challenge nc333"
author: capitol
category: ctf
---

![tent_village](/images/tent_village.jpg)

##### Name:
nc333

##### Category:
crypto

##### Points:
50

#### Writeup

This was the second challenge in the crypto category, this time a lot of different encodings.

We got [this file]({% link /assets/nc333.zip %}) that contained a large number of encodings strings
inside each other.

We wrote a small rust program to handle it:

```rust
use std::fs::File;
use std::io::prelude::*;

use base64::decode;
use crate::Commands::{Base64, Reverse, Rot13, Hex};

enum Commands {
    Base64,
    Reverse,
    Rot13,
    Hex
}

impl Commands {
    fn from(s: &str) -> Option<Commands> {
        if s.eq("base64") {
            return Some(Base64);
        }
        if s.eq("reverse") {
            return Some(Reverse);
        }
        if s.eq("rot13") {
            return Some(Rot13);
        }
        if s.eq("hex") {
            return Some(Hex)
        }

        eprintln!("unknown command: {}", s);
        None
    }
}

fn split_once(in_string: &str) -> (&str, &str) {
    let mut splitter = in_string.splitn(2, ':');
    let first = splitter.next().unwrap();
    let second = splitter.next().unwrap();
    (first, second)
}

fn rot(c :&char) -> char {
    if c.is_ascii_alphabetic() {
        let a = if c.is_ascii_lowercase() {
            b'a'
        } else {
            b'A'
        };
        let mut utf8 = [0u8; 1];
        c.encode_utf8(&mut utf8);
        let rot = (((utf8[0] - a) + 13) % 26) + a;
        std::char::from_u32(rot as u32).unwrap()
    } else {
        *c
    }
}

fn rot13(s: &String) -> String {
    s.chars().map(|c| rot(&c)).collect()
}

fn main() -> std::io::Result<()> {
    let mut file = File::open("challenge")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;

    while !contents.starts_with("BHCTF") {
        let (command, text) = split_once(&contents);
        let command = Commands::from(command).unwrap();
        let decoded = match command {
            Base64 => {
                let t = &decode(text).unwrap();
                std::str::from_utf8(t).unwrap().to_string()
            },
            Reverse => text.chars().rev().collect::<String>(),
            Rot13 => rot13(&text.to_string()),
            Hex => {
                let t = hex::decode(text).unwrap();
                std::str::from_utf8(&t).unwrap().to_string()
            }
        };
        contents = decoded;
    }
    println!("{}", contents);
    Ok(())
}
```

Flag was `BHCTF{b4se64_is_n0t_crypt0}`.