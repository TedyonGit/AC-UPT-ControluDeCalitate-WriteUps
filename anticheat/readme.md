# anticheat

After downloading the zip archive and endless scrolling through the 10.000 files I sorted them by file type and saw a .bin file. After a lot of tries I found out the encryption was RC4 bu that was not enough, I needed a mask key. In challenge description was mentioned vanguard so I searched about it's encryption and found a decryptor on unknowncheats forum and it seems the mask key in that post was used in this challenge. I created a python script to decrypt the .bin file using that mask key and I eneded up with a base64 text which revealed the flag.

```python
import struct
from pathlib import Path

def rc4(data, key):
    S = [i for i in range(256)]
    j = 0
    out = []
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    i = j = 0
    for char in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(char ^ S[(S[i] + S[j]) % 256])
    return bytearray(out)

KEYMASK = [0xB1, 0x54, 0x45, 0x57, 0xA7, 0xC4, 0x64, 0x2E,
           0x98, 0xD8, 0xB1, 0x1A, 0x0B, 0xAA, 0xD8, 0x8E,
           0x7F, 0x1E, 0x5B, 0x8D, 0x08, 0x67, 0x96, 0xCB,
           0xAA, 0x11, 0x50, 0x84, 0x17, 0x46, 0xA3, 0x30]

# Load .bin file
data = Path("key.bin").read_bytes()
data = data[4:]  # skip header

# Derive RC4 key (first 32 bytes XOR mask)
real_key = [data[i] ^ KEYMASK[i] for i in range(32)]
data = data[32:]

# Output file
with open("bin_decrypted.txt", "w", encoding="utf-8") as out:
    while len(data) > 0:
        block_len = struct.unpack('<L', data[:4])[0]
        data = data[4:]
        block = data[:block_len]
        data = data[block_len:]

        dec = rc4(block, real_key).decode("utf-16", errors="ignore")
        out.write(dec + "\n")

print("key.bin decrypted → see bin_decrypted.txt")
```

Script output: Y3RmezhhMTFkZWM3OTU4ODA4ZjAxNDVhYThiYjk1OGYyMzMyYTUzYjZjMjEwNzc2YWRiOTI2NDczOGI5YTMxZjY1Y2Z9
Decoded base64: ctf{8a11dec7958808f0145aa8bb958f2332a53b6c210776adb9264738b9a31f65cf}


# Hidden in the Cartridge

I used strings command and it revealed some game logs and at the end of each log there was hex data, I removed the junk "$$$" and it converted it to ascii characters to get the flag.

Filtered data
```
6374667b3966316234333831
613563363666633064363139
-p356235333838626565643839
66323538323234387d
```

HEX to ascii
```
ctf{9f1b4381a5c66fc0d6195b5388beed89f2582248}
```
