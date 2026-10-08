# Random-ish Writeup
# Crypto(Med)

I started by looking at the given Python code. The important part was the random number generator:

self.state = (self.a * self.state + self.c) % self.modulus

This is a Linear Congruential Generator (LCG), which means the values are predictable if we have enough consecutive outputs.

The program gave these six outputs:

291473454
604141018
2683955372
947114353
1563186696
3670295563

The modulus was:

3873239791

I used the differences between consecutive outputs and the LCG formula to recover the two unknown values, `a` and `c`.

The recovered values were:

a = 1940768799
c = 551749767

The program then generates one more value using:

next = (a * current + c) % modulus

Using the sixth output, I calculated the next value:

key = 385581677

This value is used as the XOR key.

The program creates the ciphertext using:

flag_int = int.from_bytes(FLAG, byteorder="big")
ciphertext = flag_int ^ key

So I reversed the XOR operation using:

flag_int = ciphertext ^ key

and converted the resulting integer back into bytes to recover the flag.

The main trick was realizing that the generator was not actually random. Since it was an LCG, the six given outputs were enough to recover its parameters and predict the seventh output.

Recovered values:

a = 1940768799
c = 551749767
modulus = 3873239791
key = 385581677

flag-bi0s{l1n3ar_but_n0t_s3cur3}