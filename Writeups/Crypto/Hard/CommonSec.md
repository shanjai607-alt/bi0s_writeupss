# Common Secret — CTF Writeup
# Crypto(Hard)

## Challenge summary

The challenge says the same message was encrypted twice with different keys but the same modulus. The files include a Python source file and an `output.txt` file with two ciphertexts and their public keys.

The encryption is RSA. The weakness is that both encryptions use the same modulus `n` and the same original message, but different exponents `e1` and `e2`.

Since the exponents are coprime, I can use the Extended Euclidean Algorithm to find two numbers, `a` and `b`, that satisfy:

```text
a × e1 + b × e2 = 1
```

That gives me a way to combine the ciphertexts and recover the original message.

## Step 1: Check the files

First, I listed the files in the challenge folder:

```bash
find "/Users/shanjai/Downloads/Common_Secret-5" -maxdepth 3 -type f -print
```

I found:

```text
README.md
src/challenge.py
src/output.txt
```

Then I read the files to understand how the challenge worked and where the encrypted values were stored:

```bash
cat "/Users/shanjai/Downloads/Common_Secret-5/README.md" \
    "/Users/shanjai/Downloads/Common_Secret-5/src/challenge.py" \
    "/Users/shanjai/Downloads/Common_Secret-5/src/output.txt"
```

The source showed that the program uses RSA encryption:

```text
ciphertext = message^e mod n
```

The output file gave me two ciphertexts, `c1` and `c2`. Both public keys use the same modulus `n`, but have different exponents.

## Step 2: Apply the common-modulus attack

The two encryptions can be written like this:

```text
c1 = m^e1 mod n
c2 = m^e2 mod n
```

I used the Extended Euclidean Algorithm to find values for `a` and `b` that satisfy:

```text
a × e1 + b × e2 = 1
```

For this challenge, I got:

```text
e1 = 626033
e2 = 673093
a  = 99219
b  = -92282
```

I checked the result:

```text
99219 × 626033 + (-92282) × 673093 = 1
```

With those values, I can combine the ciphertexts like this:

```text
c1^a × c2^b mod n = m
```

The value of `b` is negative, which means Python needs to calculate the modular inverse of `c2`. Python’s three-argument `pow()` can do that as part of the calculation.

## Step 3: Recover the message

I used this Python script to read the ciphertexts and public keys, check that the modulus is shared, calculate `a` and `b`, and turn the recovered number back into text:

```bash
python3 - <<'PY'
from pathlib import Path
import re

output_file = Path(
    "/Users/shanjai/Downloads/Common_Secret-5/src/output.txt"
)
data = output_file.read_text()

# Pull out each ciphertext and its public key (n, e).
rows = re.findall(
    r"Ciphertext \d+ : (\d+)\nPublic Key : \((\d+), (\d+)\)",
    data
)

if len(rows) != 2:
    raise ValueError("I expected to find two ciphertexts and two public keys")

(c1, n1, e1), (c2, n2, e2) = [
    tuple(map(int, row)) for row in rows
]

# Both keys need to use the same modulus for this attack.
if n1 != n2:
    raise ValueError("The public keys do not share a modulus")

n = n1

# Find a and b such that a*x + b*y equals the greatest common divisor.
def extended_gcd(x, y):
    if y == 0:
        return x, 1, 0

    gcd_value, a1, b1 = extended_gcd(y, x % y)
    return gcd_value, b1, a1 - (x // y) * b1

gcd_value, a, b = extended_gcd(e1, e2)

if gcd_value != 1:
    raise ValueError("The exponents are not coprime")

print("Same modulus:", n1 == n2)
print("gcd(e1, e2):", gcd_value)
print("Values of a and b:", a, b)
print("Check:", a * e1 + b * e2)

# Python handles the negative exponent using a modular inverse.
message_number = (pow(c1, a, n) * pow(c2, b, n)) % n

# Convert the recovered number into bytes and then readable text.
message_bytes = message_number.to_bytes(
    (message_number.bit_length() + 7) // 8,
    "big"
)

print("Recovered message:")
print(message_bytes.decode())
PY
```

The checks confirmed that the moduli matched and that the exponents were coprime:

```text
Same modulus: True
gcd(e1, e2): 1
Values of a and b: 99219 -92282
Check: 1
```

After that, the script printed the recovered message.

## Why this worked

Both ciphertexts were made from the same message and the same RSA modulus. Because the exponents are coprime, I could use the Extended Euclidean Algorithm to combine the ciphertexts and recover the message. I didn’t need to factor the modulus.

So, even though the message was encrypted twice, reusing the same modulus made the common-modulus attack possible.

## Flag

text
bi0s{b3z0ut_f0und_1t}


