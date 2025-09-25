# RED --- Writeup

**Writeup by:** Rasheed Gavin\
**Challenge:** RED
**Category:** Forensics / Steganography\

------------------------------------------------------------------------

## Description

An image `red.png` is provided. The challenge hint repeats **RED**
multiple times --- look for hidden data in the image.

------------------------------------------------------------------------

## Files

-   `red.png` --- challenge image

------------------------------------------------------------------------


## Solution (commands & steps)

1.  Run `zsteg` to scan the image:

``` bash
zsteg red.png
```

2.  Inspect the `zsteg` output --- one entry shows a repeated
    base64-looking string in `b1,rgba,lsb,xy`:

```{=html}
<!-- -->
```
    cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==

3.  Decode the base64 string:

``` bash
echo "cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==" | base64 -d
```

4.  Observed output (the flag):

```{=html}
<!-- -->
```
    picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}

------------------------------------------------------------------------

## Flag

`picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`

------------------------------------------------------------------------

## Explanation / Analysis

-   `zsteg` inspects PNG color channels and LSB planes. The
    `b1,rgba,lsb,xy` result contained a base64 string repeated multiple
    times --- a common pattern when a short hidden payload is embedded
    across pixels.\
-   Decoding the base64 yields the picoCTF flag in plain text.\
-   `zsteg` also reported other artifacts (OpenPGP key material, small
    text), which were not required for this challenge but indicate the
    image had multiple embedded layers or noise.

------------------------------------------------------------------------


## Lessons learned

-   Always run `zsteg` (or similar) on suspicious PNGs; LSB
    steganography is common in CTFs.\
-   Look for repeating patterns and base64-like strings --- they decode
    easily to flags.\
-   Images can contain multiple hidden payloads across different
    channels.
