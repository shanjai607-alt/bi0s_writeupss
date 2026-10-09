# Random-ish
# Crypto(Med)

I looked at the generator and noticed it was a Linear Congruential Generator (LCG):

```text
next = (a × state + c) mod modulus
```

The challenge gave me six consecutive outputs and the modulus. Although the values looked random, an LCG follows a predictable pattern. I used the differences between consecutive outputs to recover its two unknown values:

```text
a = 1940768799
c = 551749767
```

Then I used the sixth output as the current state and applied the formula once more:

```text
key = (a × 3670295563 + c) mod 3873239791
    = 385581677
```

The program had XORed the flag, converted to an integer, with this key. XOR is reversible: applying the same key again gives back the original integer. I XORed the ciphertext with `385581677` and converted the result back into bytes to get the flag.

The main trick was realizing the “random” values came from an LCG, so I could recover its settings and predict the next value.

**Recovered values:**

```text
a       = 1940768799
c       = 551749767
modulus = 3873239791
key     = 385581677
```

**Flag:** `flag-bi0s{l1n3ar_but_n0t_s3cur3}`