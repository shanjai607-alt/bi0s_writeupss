# Under a Red Sun — Writeup
# Forensics(Easy)

## Challenge idea

The challenge gives us a sunset image and mentions a “red sun.” That suggests looking at the image’s color channels. A color image stores separate values for red, green, and blue at each pixel. Hidden text can be stored in those values without being visible in the picture.

I used `zsteg`, a tool that checks image channels and bit patterns for hidden data.

## Steps

First, I checked the file type and its metadata. It is an RGB PNG image. The metadata check did not reveal the flag, and `binwalk` did not report an embedded archive or other file.

Then I ran `zsteg -a`, which tries many ways of reading hidden data from the image. Its output included this result:

```text
b8,r,msb,xy: "bi0s{13t_R3d_5un_5h1n3_Up0n_U5}"
```

The result tells us how the data was found:

- `b8` means zsteg reads all 8 bits in each value.
- `r` means it reads the red channel.
- `msb` means it reads the most significant bit first.
- `xy` is the pixel scan order.

I then ran a focused scan using those settings. The flag appeared as readable text.

## Commands

Run these commands in Terminal:

```bash
cd ~/Downloads
file RedSun.png
exiftool -a -u -g1 RedSun.png
binwalk RedSun.png
zsteg -a RedSun.png
zsteg -l 128 -b 8 -c r --msb -o xy RedSun.png
```

The last command checks the first 128 bytes using the red channel, reads all 8 bits, and prints any text it finds. Its output is:

```text
b8,r,msb,xy: "bi0s{13t_R3d_5un_5h1n3_Up0n_U5}"
```

## Flag

```text
bi0s{13t_R3d_5un_5h1n3_Up0n_U5}
```