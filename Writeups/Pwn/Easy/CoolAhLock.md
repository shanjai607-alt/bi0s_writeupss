# COOL AHH LOCK - PWN WRITEUP
# Pwn(Easy)

#1

I downloaded the challenge files and opened the terminal in the challenge directory:

cd ~/Downloads/cool_ahh_lock

I checked the files:

ls -l

The important files were:
- cool_ahh_lock       -> compiled binary
- cool_ahh_lock.c     -> C source code
- flag.txt            -> flag file

I ran the binary:

./cool_ahh_lock

The program gives us a 4-digit number lock and asks which position we want to change.

#2

The source contains:

int isadmin = 0;
int lock[4] = {0, 0, 0, 0};
int password[4] = {0, 0, 0, 0};

The password is randomly generated, so we don't know the correct 4 digits.

The program checks the user's chosen position using:

scanf("%d", &idx);

if (idx > 4) {
    puts("theres only 4 numbers to be set :/ try again");
    continue;
}

scanf("%d", &lock[idx - 1]);

Normally:

idx = 1  -> lock[0]
idx = 2  -> lock[1]
idx = 3  -> lock[2]
idx = 4  -> lock[3]

However, the program only checks whether idx is greater than 4.

It does NOT check whether idx is less than 1.

Therefore, negative values are accepted.

For example:

idx = -3

Then:

idx - 1 = -4

So the program accesses:

lock[-4]

This is outside the lock array and is therefore an OUT-OF-BOUNDS WRITE.

#3 

The program also contains:

int isadmin = 0;

Later it checks:

if (isadmin) {
    theSecretChamber();
}

The secret function is:

void theSecretChamber() {
    system("cat flag.txt");
}

Therefore, instead of trying to guess the random password, we can try to modify isadmin.

The goal becomes:

Out-of-bounds write
        ↓
Modify isadmin
        ↓
isadmin becomes non-zero
        ↓
theSecretChamber()
        ↓
cat flag.txt
        ↓
FLAG

#4 Using Gdb
I opened the binary with GDB:

gdb ./cool_ahh_lock

GDB is the GNU Debugger. It allows us to inspect the program and its memory.

Since the original binary did not provide the debugging symbols needed to directly inspect variables, I compiled the provided C source with debugging information:

gcc -g -O0 cool_ahh_lock.c -o cool_ahh_lock_debug

Then:

gdb ./cool_ahh_lock_debug

Inside GDB, I checked the memory addresses of the important variables:

p &isadmin

p &lock

This allowed me to determine the relative position of isadmin and the lock array in memory.

#5 exploit

The required negative index was:

-3

The program then asks for the value to write.

I entered:

1

Then:

n

So the final input was:

-3
1
n

The calculation is:

idx = -3
idx - 1 = -4

Therefore the program performs:

lock[-4] = 1

Because lock[-4] is outside the array, this out-of-bounds write reaches the memory location of isadmin.

This changes:

isadmin = 0

to:

isadmin = 1

The program then executes:

theSecretChamber();

which runs:

system("cat flag.txt");

and displays the flag.

#6 Commands used:

cd ~/Downloads/cool_ahh_lock
ls -l
./cool_ahh_lock
gdb ./cool_ahh_lock
gcc -g -O0 cool_ahh_lock.c -o cool_ahh_lock_debug
gdb ./cool_ahh_lock_debug

Inside GDB:

p &isadmin
p &lock

#7 Final Exploit Input

-3
1
n

#8 Vulnerability

The vulnerability is an out-of-bounds array write caused by incomplete bounds checking.

The program checks:

if (idx > 4)

but fails to check:

if (idx < 1)

This allows negative indexes such as -3, which can write to memory outside the lock array and overwrite isadmin.

#9 Attack FLow

The attack flow begins by bypassing the need to guess a random password through source code inspection, which reveals a missing lower-bound check. This allows the use of a negative index, specifically lock[-4], to overwrite the isadmin variable and set it to 1. Finally, this grants access to execute theSecretChamber() and retrieve the flag by reading the flag.txt file.