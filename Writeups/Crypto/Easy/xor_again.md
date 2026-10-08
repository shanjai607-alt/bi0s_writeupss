# XOR Again — Writeup

# Crypto(Easy)

The challenge uses a repeating three-byte XOR key:

```python
ciphertext = bytes(p ^ k for p, k in zip(plaintext, cycle(key)))
```

The flag format is `bi0s{...}`, so the first three plaintext bytes are known. XORing them with the first three ciphertext bytes reveals the key:

```text
0x25 XOR 'b' = 0x47 = 'G'
0x0e XOR 'i' = 0x67 = 'g'
0x05 XOR '0' = 0x35 = '5'
```

The key is `Gg5`. XOR each ciphertext byte with the repeating key to recover the flag.

## Commands

Run these commands in Terminal:

```bash
cd ~/Downloads/XOR_Again-2
find . -maxdepth 3 -type f -print
cat README.md
sed -n '1,200p' src/challenge.py
cat src/output.txt

python3 - <<'PY'
from hashlib import md5

ciphertext = bytes.fromhex(
    "250e05341c412f564618564618105d3e384274385177091233384774124674385e741e463a"
)
known_prefix = b"bi0s{"

key = bytes(ciphertext[i] ^ known_prefix[i] for i in range(3))
plaintext = bytes(
    byte ^ key[i % len(key)]
    for i, byte in enumerate(ciphertext)
)

assert bytes(
    byte ^ key[i % len(key)]
    for i, byte in enumerate(plaintext)
) == ciphertext

print("Key:", key.decode())
print("Flag:", plaintext.decode())
print("MD5:", md5(plaintext).hexdigest())
PY
```

## Result

```text
Key: Gg5
Flag: bi0s{th1s_1s_why_w3_d0n't_r3us3_k3ys}
```

## Hash note

The challenge page shows the MD5 `8dbccca79b7d9ca60904644aa3d2088a`. The recovered flag’s MD5 is `a5ef189b465fbd9d778d73bd89ca5d96`, so the displayed hash does not match the attached ciphertext’s decrypted plaintext.