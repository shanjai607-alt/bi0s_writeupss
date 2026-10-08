# i_hate_z3 — Writeup

**Category:** Reverse Engineering  
**Difficulty:** Easy

## Analysis

The program reads the flag as bytes and rejects anything that is not exactly 24 bytes long. It then XORs every byte with `0xff` and checks the high and low nibbles of each result.

Together, those nibble checks fix every transformed byte. The targets are:

```text
9d 96 cf 8c 84 96 a0 9e 9c 8b 8a 9e
93 93 86 a0 93 90 89 9a a0 85 cc 82
```

For each input byte `b`, the constraint is `b XOR 0xff = target`. Since XOR with `0xff` flips every bit, XORing each target with `0xff` recovers the input byte. The SMT model below encodes those constraints directly.

## Commands

Run these commands from the directory containing `chall.py`:

```bash
cd ~/Downloads
sed -n '1,240p' chall.py
python3 -m venv .venv
source .venv/bin/activate
python -m pip install z3-solver

cat > solve.py <<'PY'
from z3 import BitVec, Solver, sat

targets = bytes.fromhex(
    "9d96cf8c8496a09e9c8b8a9e939386a09390899aa085cc82"
)
assert len(targets) == 24

flag_bytes = [BitVec(f"flag_{i}", 8) for i in range(len(targets))]
solver = Solver()

for byte, target in zip(flag_bytes, targets):
    solver.add((byte ^ 0xff) == target)

if solver.check() != sat:
    raise SystemExit("No solution")

model = solver.model()
flag = bytes(
    model.eval(byte, model_completion=True).as_long()
    for byte in flag_bytes
)
print(flag.decode())
PY

python solve.py
printf '%s\n' 'bi0s{i_actually_love_z3}' | python3 chall.py
```

The solver prints the recovered flag, and the final command checks it against the supplied challenge script.

## Flag

```text
bi0s{i_actually_love_z3}
```