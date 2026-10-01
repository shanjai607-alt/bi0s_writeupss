GDB_TUTORIAL Writeup

I started by checking the files given in the challenge. The important files were `enc`, `prog`, `run.sh` and `exit_chall.sh`.

First, I gave the scripts execute permission and ran the provided script:

chmod +x run.sh exit_chall.sh
./run.sh

The `run.sh` script decodes the `enc` file using XOR with 0xCC and then starts GDB with `prog`. The decoded file is loaded through `.gdbinit`.

To find the password and salt, I checked the functions inside `prog` using:

objdump -d -M intel prog

The two interesting functions were `gen_pass` and `gen_salt`.

`gen_pass` contains a 16-byte array which is XORed with `(index % 4)`. After applying the XOR operation, the bytes convert to the following string:

0hhhh_y0u_g0t_1t

So the password is:

0hhhh_y0u_g0t_1t

The `gen_salt` function uses the same XOR operation, but instead of creating a string, it adds the resulting values together. The final value is:

1467

So the salt is:

1467

The decoded Python script showed that these two values are used with PBKDF2 to generate a 32-byte AES key. The ciphertext is then decrypted using AES-256-CBC.

The challenge also tells us to run:

found_pass

inside GDB, which asks for the password and salt.

Using:

Password: 0hhhh_y0u_g0t_1t
Salt: 1467

the ciphertext decrypts successfully and gives the flag:

bi0s{1mpr3ss3d_1f_n0_chatgpt}
