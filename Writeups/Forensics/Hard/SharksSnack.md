## Shr3k’s Snacks
# Forensics(Hard)

I started by checking the chat traffic in the capture. It said the secret was hidden in an image, and the Shrek hint gave me a likely password: `1_am_$hrek`. I then followed the HTTP upload in Wireshark and saved the JPEG.

I used these commands to read the chat and extract the hidden data:

```bash
cd ~/Downloads/Shr3k_1s_trav3ll1ng_handout
tcpdump -nn -A -r chall.pcapng 'tcp port 1234'
steghide extract -sf shrek.jpg -p '1_am_$hrek'
ls -l
```

The single quotes around the password matter: they stop the `$` from being treated as a shell variable. After extraction, I checked the output file and read it:

```bash
file extracted_file
cat extracted_file
```

That revealed the flag...

**Flag:** `BI0S(5HR3K-15-H4PPYLY-5H4R1NG-H1S-53CRET-5N4CK5-W1TH-Y0U!!)`