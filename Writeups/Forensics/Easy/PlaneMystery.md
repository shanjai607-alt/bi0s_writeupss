# A Plane Ol’ Mystery — Writeup
# Forensics(Easy)

**Category:** Forensics  
**Difficulty:** Easy

## My approach

I read the title, **A Plane Ol’ Mystery**, and noticed the word “Plane.” The image also says “Details,” which made me think the message might be hidden in an image bit plane.

A PNG stores each pixel as red, green, and blue values. Each value has 8 bits. A bit plane shows one selected bit from every pixel as black or white. A hidden message can appear in one of these planes even when it is hard to see in the original image.

## What I found

I first checked the file type, metadata, and whether another file was attached to the end of the PNG. Those checks showed a regular RGBA PNG and didn’t reveal the message. I also ran `zsteg -a`, but it didn’t show the flag as readable text.

Next, I used Python and Pillow to make a contact sheet of the red, green, and blue bit planes. I found the handwriting in **Green bit 1**. Since bit counting starts at 0, bit 1 is the second-lowest bit of each green value. I saved that plane as a larger image so I could read it more easily.

The message says `bi0s{1n_pl4in_s1ght}`, which reads as “in plain sight.”

