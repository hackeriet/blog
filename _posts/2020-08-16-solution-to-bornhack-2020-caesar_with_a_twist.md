---
layout: post
title: "Solution to Bornhack 2020 CTF challenge caesar_with_a_twist"
author: capitol
category: ctf
---

![caesar](/images/caesar.jpg)

##### Name:
caesar_with_a_twist

##### Category:
crypto

##### Points:
75

#### Writeup

The third challenge in the crypto category, this was the first one with some actual encryption.

We got an encrypted text:

```
CLLJE{QtLo_wF_r_YqOI_bXrH_gpJw_WxPh_Sm_QraVmo_Se_IoSvvn_XyR_GeU_Ack_EDRDTIosscf_rj}
```

And a Python program that described how it was generated, analysing that program showed that
it was a regular substitution cipher where the substitution key was changed for each character.
The key was a function of that characters position in the string.

We reimplemented it in rust and wrote a reverse implementation:

```rust
use std::fs::File;
use std::io::prelude::*;

#[derive(Debug, Clone)]
struct CaesarError;

fn caesar_encrypt(original: char, base: u32) -> Result<char, CaesarError> {
    let letter = original as u32;

    // Ensures the base rotation does not exceed 26
    // If the base is 28, it exceeds 26, and ends up being 2 (after modulus)
    let base = base % 26;

    // If the letter is a space, underscore or curly brackets, just return it without rotating
    if letter == 32 || letter == 95 || letter == 123 || letter == 125 {
        return Ok(std::char::from_u32(letter).unwrap());
    }

    // If capital letter
    if 'A' as u32 <= letter && letter <= 'Z' as u32 {
        // If the base exceeds the alphabet
        return if ('Z' as u32) < (letter + base) {
            Ok(std::char::from_u32(letter + base - 26).unwrap())
        } else {
            Ok(std::char::from_u32(letter + base).unwrap())
        }
    }

    // If non-capital letter
    if 'a' as u32 <= letter && letter <= 'z' as u32 {
        // If the base exceeds the alphabet
        return if ('z' as u32) < (letter + base) {
            Ok(std::char::from_u32(letter + base - 26).unwrap())
        } else {
            Ok(std::char::from_u32(letter + base).unwrap())
        }
    }

    Err(CaesarError {})
}

fn caesar_decrypt(original: char, base: u32) -> Result<char, CaesarError> {
    let letter = original as u32;

    // Ensures the base rotation does not exceed 26
    // If the base is 28, it exceeds 26, and ends up being 2 (after modulus)
    let base = base % 26;

    // If the letter is a space, underscore or curly brackets, just return it without rotating
    if letter == 32 || letter == 95 || letter == 123 || letter == 125 {
        return Ok(std::char::from_u32(letter).unwrap());
    }

    // If capital letter
    if 'A' as u32 <= letter && letter <= 'Z' as u32 {
        // If the base exceeds the alphabet
        return if ('A' as u32) > (letter - base) {
            Ok(std::char::from_u32(letter - base + 26).unwrap())
        } else {
            Ok(std::char::from_u32(letter - base).unwrap())
        }
    }

    // If non-capital letter
    if 'a' as u32 <= letter && letter <= 'z' as u32 {
        // If the base exceeds the alphabet
        return if ('a' as u32) > (letter - base) {
            Ok(std::char::from_u32(letter - base + 26).unwrap())
        } else {
            Ok(std::char::from_u32(letter - base).unwrap())
        }
    }

    Err(CaesarError {})
}

fn encrypt(s: &String) -> Result<String, CaesarError> {
    let mut result:Vec<char> = vec![];
    for (i, c) in s.chars().enumerate() {
        result.push(caesar_encrypt(c, ((i + 1) * (i + 1)) as u32)?);
    }
    Ok(result.iter().collect::<String>())
}

fn decrypt(s: &String) -> Result<String, CaesarError> {
    let mut result:Vec<char> = vec![];
    for (i, c) in s.chars().enumerate() {
        result.push(caesar_decrypt(c, ((i + 1) * (i + 1)) as u32)?);
    }
    Ok(result.iter().collect::<String>())
}

fn main() -> std::io::Result<()> {
    let mut file = File::open("cwat_encrypted_flag.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;

    println!("{}", decrypt(&contents).unwrap());
    Ok(())
}

#[cfg(test)]
mod tests {
    use crate::CaesarError;
    use crate::caesar_decrypt;
    use crate::caesar_encrypt;
    use crate::encrypt;
    use crate::decrypt;

    static ASCII_LOWER: [char; 26] = [
        'a', 'b', 'c', 'd', 'e',
        'f', 'g', 'h', 'i', 'j',
        'k', 'l', 'm', 'n', 'o',
        'p', 'q', 'r', 's', 't',
        'u', 'v', 'w', 'x', 'y',
        'z',
    ];

    static ASCII_UPPER: [char; 26] = [
        'A', 'B', 'C', 'D', 'E',
        'F', 'G', 'H', 'I', 'J',
        'K', 'L', 'M', 'N', 'O',
        'P', 'Q', 'R', 'S', 'T',
        'U', 'V', 'W', 'X', 'Y',
        'Z',
    ];

    #[test]
    fn loop_all_ascii_chars() -> Result<(), CaesarError> {
        for base in 0..25 {
            println!("base = {}", base);
            for c in &ASCII_LOWER {
                assert_eq!(*c, caesar_decrypt(caesar_encrypt(*c, base)?, base)?);
            }
            for c in &ASCII_UPPER {
                assert_eq!(*c, caesar_decrypt(caesar_encrypt(*c, base)?, base)?);
            }
        }

        Ok(())
    }

    #[test]
    fn encrypt_test() -> Result<(), CaesarError> {
        assert_eq!("Ulri sp d ksfh", encrypt(&"This is a test".to_string())?);

        Ok(())
    }

    #[test]
    fn encrypt_loop() -> Result<(), CaesarError> {
        assert_eq!("This is a test", decrypt(&encrypt(&"This is a test".to_string())?)?);

        Ok(())
    }
}
```

Flag was `BHCTF{ThIs_iS_a_VeRY_lOnG_flAg_MaDe_By_CaeSar_To_EnSure_YoU_DiD_Not_BRUTUSforce_it}`.