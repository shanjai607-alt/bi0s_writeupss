# Poison Dart — Writeup
# AI(Hard)

This challenge was about tricking PetSnap’s cat-or-frog model. The model keeps its image feature extractor frozen, but retrains a small final layer using the photos uploaded that day.

I made five slightly modified frog images and used them as training examples. The goal was to make the model treat the target cat as a frog while keeping the changes subtle and the rest of the model’s accuracy intact. Each colour channel was allowed to change by at most 16, so the modified images still looked like the originals.

I checked my images with the provided checker. It accepted all five, classified the target as a frog with 99.8% confidence, and showed that overall accuracy stayed at 93.70%. After the check passed, I sent the images to the organizers and got the flag from a club member.

## Commands I used

These are the commands shown in my terminal while I copied and checked the images:

```bash
ls -lh poison_*.png
cp poison_*.png ~/petsnap/
cd ~/petsnap
ls -lh poison_*.png
python3 challenge.py poison_0.png poison_1.png poison_2.png poison_3.png poison_4.png
```

The checker’s `PASS` meant the target had flipped to frog and the overall accuracy stayed above the allowed minimum.

## Flag

```text
bi0s{p0is0n_dar7_n0t_g00d_f0r_h3alth}
```
