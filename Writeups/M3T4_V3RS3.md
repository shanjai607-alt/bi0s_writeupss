# M3T4_V3RS3 — Writeup

**Category:** Forensics  
**Difficulty:** Easy

## Analysis

The description emphasizes “EX” and “IF,” which together point to **EXIF** metadata. The challenge title also hints at metadata. Since the attachment is a PNG image, inspect its metadata with ExifTool.

ExifTool reveals a custom PNG metadata field named `Fl 4g` containing the flag. The image’s MD5 also matches the value in the README, confirming this is the expected file.

## Commands

Run these commands in Terminal:

```bash
cd ~/Downloads/M3T4_V3RS3_handout
find . -maxdepth 4 -type f -print
cat Readme.md
file M3t4_chall.png
exiftool -a -u -g1 M3t4_chall.png
md5 M3t4_chall.png
```

The metadata output includes:

```text
Fl 4g : bi0s{y3y!!_y0u_f0und_m3t4d4t4_us1ng_exift00l}
```

The MD5 command returns `10d1caa52b5af7b8b0904373ef816422`, matching the hash listed in the README.

## Flag

```text
bi0s{y3y!!_y0u_f0und_m3t4d4t4_us1ng_exift00l}
```