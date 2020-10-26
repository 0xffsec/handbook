---
title: Steganography
weight: 202
---

# Steganography

## At a Glance

Steganography is the art and science
of hiding a message,
image,
or file
within another message,
image,
or file
[^steganography-wiki]

{{<note>}}
Steganography is used to hide
the occurrence of communication.
{{</note>}}

## General

Determine the file type.

```sh
file {{< param "war.tfile" >}}
```

Read file meta-data.

```sh
exiftool {{< param "war.tfile" >}}
```

Extract printable characters.

```sh
strings -n 6 -e s {{< param "war.tfile" >}}
```
{{<details "Parameters">}}
- `-n <length>`: Print sequence of at least `<length>` chars long.
- `-e <encoding>`: Character encoding of the strings that are to be found.
    - `s`: single-7-bit-byte characters (ASCII, ISO 8859, etc., default).
    - `S`: single-8-bit-byte characters.
    - `b`: 16-bit bigendian.
    - `l`: 16-bit littleendian.
    - `B`: 32-bit bigendian.
    - `L`: 32-bit littleendian.
{{</details>}}

## Text

Look for anomalies in font and spacing.

[Unicode Text Steganography Encoders / Decoders](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder)

## Images

### steghide [^steghide]

Steghide supports **JPEG**, **BMP**, **WAV** and **AU** file formats.

Display information about a cover or stego file.

```sh
steghide info {{< param "war.tfile" >}}
```

Extract secret data.

```sh
steghide extract -sf {{< param "war.tfile" >}}
```

{{<details "Parameters">}}
- `info`: Display information about a cover or stego file.
- `extract`: Extract secret data from a stego file.
- `-sf <file>`:  Specify the name for the stego file.
{{</details>}}

### StegoVeritas

[StegoVeritas](https://github.com/bannsec/stegoVeritas) is a powerful multi-tool.
Supports **GIF**, **JPEG**, **PNG**, **TIFF**, **BMP** file formats
and will attempt to
run on any file.

```sh
stegoveritas {{< param "war.tfile" >}}
```

### LSBSteg [^stegolsb]

[LSBSteg](https://github.com/ragibson/Steganography#lsbsteg) uses LSB steganography
to hide and recover files
from the color information
of an RGB image,
**BMP** or **PNG**.

```sh
stegolsb steglsb -r -i {{< param "war.tfile" >}} -o output.zip -n 2
```
{{<details "Parameters">}}
- `-r`: To recover data from a sound file.
- `-i <file>`: Path to `.wav` file.
- `-o <file>`: Path to an output file.
- `-n <int>`: LSBs to use (default: 2).
{{</details>}}

### Online Tools

- [Steganographic Decoder](https://futureboy.us/stegano/decinput.html) - Steghide
- [Forensically](https://29a.ch/photo-forensics/) - Digital image forensics multi-tool.
- [Magic Eye Solver](https://magiceye.ecksdee.co.uk/)

## Audio

See [steghide](#steghide-steghide)

### WavSteg [^stegolsb]

[WavSteg](https://github.com/ragibson/Steganography#WavSteg) uses LSB steganography
to hide and recover files
from the samples of a **WAV** file.

```sh
stegolsb wavsteg -r -i {{< param "war.tfile" >}} -o output.txt -n 2 -b 5589889
```
{{<details "Parameters">}}
- `-r`: To recover data from a sound file.
- `-i <file>`: Path to `.wav` file.
- `-o <file>`: Path to an output file.
- `-n <int>`: LSBs to use (default: 2).
- `-b <int>`: How many bytes to recover from the sound file.
{{</details>}}

## Sonic Visualiser

[Sonic Visualizer](https://www.sonicvisualiser.org/) is an application
for viewing and analysing
the content of music audio files.

### DTMF

[DTMF Decoder](https://unframework.github.io/dtmf-detect/)

## Further Reading

- [How to Hide Secret Messages in Music Files](https://www.iicybersecurity.com/audio-steganography.html)
- [Detecting Steganography Content on the Internet](http://niels.xtdnet.nl/papers/detecting.pdf)

[^steganography-wiki]: “Steganography - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 31 Oct. 2001, https://en.wikipedia.org/wiki/Steganography.
[^steghide]: Hetzl, Stefan. “Manual.” Steghide, http://steghide.sourceforge.net/documentation/manpage.php.
[^stegolsb]: ragibson. “Ragibson/Steganography: Least Significant Bit Steganography.” GitHub, https://github.com/ragibson/Steganography.
