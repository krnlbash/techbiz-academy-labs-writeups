# Lab 16 — Cryptography & Encoding: Telling Them Apart

**Category:** Cryptography Fundamentals
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- List the sample strings
- Identify and decode the Base64 string
- Identify and decode the hex string
- Break the Caesar cipher
- Identify the hash (and see why it will not decode)
- Record the encoding vs. encryption distinction

## Methodology

### 1. List samples
```
samples
```
```
VGVjaEJpeg==
5442537b656e636f64696e675f69735f6e6f745f656e6372797074696f6e7d
Grpu Ovm Frphevgl
5f4dcc3b5aa765d61d8327deb882cf99
```
Four distinct sample types requiring different handling: Base64, hexadecimal, a Caesar cipher, and an MD5 hash.

### 2. Base64 sample
```
identify VGVjaEJpeg==
decode VGVjaEJpeg==
```
```
Base64 — ends with = padding, alphabet A-Za-z0-9+/. ENCODING, not encryption:
reversible by anyone, no key involved.
Decoded : TechBiz
```

### 3. Hexadecimal sample
```
identify 5442537b656e636f64696e675f69735f6e6f745f656e6372797074696f6e7d
decode 5442537b656e636f64696e675f69735f6e6f745f656e6372797074696f6e7d
```
```
Hexadecimal — only 0-9 and a-f, even length. ENCODING: each byte written as two characters.
Decoded : TBS{encoding_is_not_encryption}
```

### 4. Caesar cipher
```
rot13 Grpu Ovm Frphevgl
```
```
ROT13 : Tech Biz Security
>> A Caesar cipher has no key worth the name — 25 shifts is all there is.
```

### 5. Hash sample
```
hashid 5f4dcc3b5aa765d61d8327deb882cf99
```
```
Most likely: MD5 (32 hex characters)
hashcat mode: 0
>> A hash cannot be reversed. It is cracked by hashing guesses until one matches.
```

### 6. Record the distinction
```
concept
```

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Is Base64 encryption? | `no` |
| q2 | What the Base64 sample decodes to | `TechBiz` |
| q3 | Places ROT13 shifts each letter | `13` |
| q4 | Why a hash cannot be decoded back | `one way` |
| q5 | Flag (from decoding the hex sample) | `TBS{encoding_is_not_encryption}` |

## Takeaways
- **Encoding** (Base64, hex, URL-encoding) is a reversible transformation for representing data in a different format — anyone can decode it, no key required. It provides *zero* confidentiality and should never be treated as a security control.
- **Encryption** requires a key; without it, ciphertext cannot practically be reversed (for strong algorithms). A Caesar cipher is a toy exception — with only 25 possible shifts, it's trivially brute-forced and offers no real security either, useful mainly as a teaching example of *weak* "encryption."
- **Hashing** is fundamentally different from both: it's a one-way function, deliberately non-reversible by design. There is no decode operation — the only way to "recover" the input from a hash is to guess candidate inputs, hash each one, and check for a match (cracking), which is why hash speed and salting matter so much for password storage.
- Conflating these three (calling Base64 "encryption," assuming a hash can be "decrypted") is one of the most common and consequential misunderstandings in security — it leads to false confidence that data is protected when it is merely encoded or hashed inappropriately for the use case.
