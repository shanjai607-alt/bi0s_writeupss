## Scaled Secrets
# Crypto(Med)

I checked the challenge files and noticed the same flag had been multiplied by a known number before each RSA encryption. Since all five ciphertexts used the small exponent `e = 3`, I could remove those known multipliers, combine the results with the Chinese Remainder Theorem, and take an exact cube root to recover the flag.

Here are the commands and the recovery script I used:

```bash
cd ~/Downloads/Scaled_Secrets-2
find . -maxdepth 3 -type f -print
cat README.md src/challenge.py src/output.txt

python3 - <<'PY'
import ast
from pathlib import Path

# Read the five ciphertexts, moduli and exponents.
rows = []
for line in Path("src/output.txt").read_text().splitlines():
    _, values = line.split(": ", 1)
    rows.append(ast.literal_eval(values))

multipliers = [2, 5, 7, 11, 13]
pad = int.from_bytes(b"nani?!", "big")

# Remove each known multiplier modulo its RSA modulus.
residues = []
product = 1
for row, multiplier in zip(rows, multipliers):
    n, c = row["n"], row["ct"]
    scale = pad * multiplier
    c_unscaled = c * pow(pow(scale, 3, n), -1, n) % n
    residues.append((c_unscaled, n))
    product *= n

# Use CRT to combine the five values of flag^3.
cube = 0
for value, n in residues:
    part = product // n
    cube += value * part * pow(part, -1, n)
cube %= product

# Find the integer cube root.
low, high = 0, 1
while high ** 3 <= cube:
    high *= 2
while low + 1 < high:
    mid = (low + high) // 2
    if mid ** 3 <= cube:
        low = mid
    else:
        high = mid

assert low ** 3 == cube
flag = low.to_bytes((low.bit_length() + 7) // 8, "big")
print(flag.decode())
PY
```

The script printed the flag directly. The key idea was that the padding wasn’t random: its multipliers were visible in the challenge source, so I could undo them before combining the ciphertexts.

**Flag:** `bi0s{ch1nese_r3m4ind3r_th0rem_f0r_th3_win}`