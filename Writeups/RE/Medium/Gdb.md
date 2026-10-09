# GDB_TUTORIAL
# RE(Med)

I started by checking the challenge files. `run.sh` decodes `enc` and starts GDB with `prog`, so I made the scripts executable and ran it:

```bash
chmod +x run.sh exit_chall.sh
./run.sh
```

I then looked through the disassembly for the password and salt functions:

```bash
objdump -d -M intel prog
```

In `gen_pass`, I found a 16-byte array. The program XORs each byte with its index modulo 4. After applying that operation, the bytes spell:

```text
0hhhh_y0u_g0t_1t
```

The `gen_salt` function uses the same XOR idea, then adds the resulting values together. That gave me the salt `1467`.

The decoded script showed that these values are used with PBKDF2 to make an AES-256 key, which decrypts the ciphertext. The challenge provides a GDB command called `found_pass`; I ran it and entered the password and salt when prompted:

```text
Password: 0hhhh_y0u_g0t_1t
Salt: 1467
```

The decryption worked and revealed the flag.

**Flag:** `bi0s{1mpr3ss3d_1f_n0_chatgpt}`