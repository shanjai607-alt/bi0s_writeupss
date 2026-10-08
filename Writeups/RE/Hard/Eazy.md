# EAZY — Writeup
# RE(Hard)

I started by checking the challenge file and saw it was a Linux executable. I searched it for clues, then looked at the `main`, `rol`, and `ror` functions. That showed me the long `EAZY?` string was a set of instructions the program used to check the flag, one character at a time.

I wrote a small Python script to try printable characters against those checks. A few positions had more than one possible answer, so I used the flag format and readable words to pick the right characters.

That gave me:

```text
bi0s{hmm_win_u_did_easy_it_was_t00_easy}
```

## Commands I used

```bash
file "/Users/shanjai/Downloads/eazy-2"
strings -n 4 "/Users/shanjai/Downloads/eazy-2"
nm -C "/Users/shanjai/Downloads/eazy-2"
objdump --disassemble-symbols=main,rol,ror --disassembler-options=intel "/Users/shanjai/Downloads/eazy-2"
objdump -s -j .rodata "/Users/shanjai/Downloads/eazy-2"
python3  # to run my character-checking script
```