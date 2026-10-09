# ASM Labs I — CTF Writeup 
# RE(Med)

## Challenge summary

The challenge asked me to solve assembly puzzles to get the flag. The attached file was a Linux program named `ASM_LABS`. I first checked what kind of file it was instead of trying to run it.

I then inspected the program and found that it contains AES-encrypted data, along with the key and IV needed to decrypt it. Decrypting the challenge’s stored ciphertext revealed the flag.

## Checked the challenge file

I opened Terminal and checked the file type:

```bash
file "/Users/shanjai/Downloads/handout-3/ASM_LABS"
```

This showed that `ASM_LABS` is a 64-bit Linux executable. Since I was on macOS, it would not run directly as a normal Mac program.

## Looked for useful text inside the program

I used `strings` to display readable text stored in the executable:

```bash
strings "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -iE 'flag|level|assembly|power|bonus|zf|nf'
```

This showed messages about assembly puzzles, the Zero Flag (`ZF`), the Sign Flag (`NF` or `SF`), and completing levels. It also showed that the program contains a bonus assembly puzzle.

## Inspected the program’s symbols and assembly references

I checked the program’s named functions for clues about how it worked:

```bash
nm -C "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -iE 'flag|aes|decrypt|snow|feeere'
```

This revealed an AES decryption function and two other functions named `snow` and `feeere`. I used `objdump` to inspect the parts of the executable that reference those functions:

```bash
objdump -d --disassembler-options=intel "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -n -B 12 -A 12 'call.*8d00'
```

The references showed that the program passes encrypted data, a key, and an IV to the AES decryption function. An **IV**, or initialization vector, is an extra value used with CBC-mode encryption.

## Found the encrypted bytes, key, and IV

I viewed the relevant data stored in the executable:

```bash
xxd -s 0x16340 -l 128 "/Users/shanjai/Downloads/handout-3/ASM_LABS"
```

The bytes near offset `0x16380` were the ciphertext used by `feeere`. The nearby readable text contained the key and IV:

- **Key:** `hehasdonitfinaly`
- **IV:** `orhashehashebeen`
- **Ciphertext location:** offset `0x16380`, length `48` bytes

I treated the ciphertext as binary data. The first `xxd` command prints it as hexadecimal text, so I used `xxd -r -p` to turn that text back into bytes before passing it to OpenSSL.

## Decrypted the ciphertext

I ran this command to decrypt the 48 bytes with AES-128-CBC:

```bash
xxd -p -s 0x16380 -l 48 "/Users/shanjai/Downloads/handout-3/ASM_LABS" \
  | tr -d '\n' \
  | xxd -r -p \
  | openssl enc -d -aes-128-cbc \
      -K 6865686173646f6e697466696e616c79 \
      -iv 6f72686173686568617368656265656e \
      -nopad \
  | tr -d '\n'
```

The `-K` and `-iv` options take hexadecimal values:

- `hehasdonitfinaly` becomes `6865686173646f6e697466696e616c79`
- `orhashehashebeen` becomes `6f72686173686568617368656265656e`

The command printed the flag:

```text
bi0s{k4444_m3333_haaaa_meeee_h4444!!!}
```

## Final flag

```text
bi0s{k4444_m3333_haaaa_meeee_h4444!!!}
```

## Commands used

Here are the commands I used during the analysis:

```bash
file "/Users/shanjai/Downloads/handout-3/ASM_LABS"

strings "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -iE 'flag|level|assembly|power|bonus|zf|nf'

nm -C "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -iE 'flag|aes|decrypt|snow|feeere'

objdump -d --disassembler-options=intel "/Users/shanjai/Downloads/handout-3/ASM_LABS" | grep -n -B 12 -A 12 'call.*8d00'

xxd -s 0x16340 -l 128 "/Users/shanjai/Downloads/handout-3/ASM_LABS"
```