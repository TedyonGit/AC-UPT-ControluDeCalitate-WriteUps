#kidnapped_by_llamas

One of the last guardians of the Ancient Egyptian Key of Life has vanished. On his way to deliver its hidden power to the alpacas, disaster struck: a herd of rebel llamas rose from the shadows, and dragged him into their forbidden kingdom.
But this mage was no ordinary alpaca. To escape certain doom, he invoked the Ancient Egyptian Key of Life and wove a spell of concealment — hiding his very presence from the llamas’ gaze. Only the worthy may now find him.
The llamas prepare their ancient sacrifice… time is running out. Your mission: decipher the encrypted trail and find the mage before the kingdom’s eternal eclipse begins.
Flag format: CTF{E:N} where E and N are the coordinates in degrees with 2 decimals (truncate, don’t round).


# EXIF UserComment (hex) → XOR with “Ankh” → coordinates → flag

The lore hint “Ancient Egyptian **Key of Life**” ⇒ **Ankh** (use as XOR key). The only odd EXIF field was **User Comment** holding hex. Hex-decode it, XOR with the repeating key `"Ankh"`, and you get `E:N` coordinates. Truncate to 2 decimals (already 2dp) and wrap.

## Steps

- `exiftool` shows `User Comment: 735c455b72545f5d6f595e`.

- Hex → bytes, XOR with `"Ankh"` (story key) → `"22.33:45.75"`.

- Format as `CTF{E:N}` with truncation (no rounding).

## Script used

`python3 - << 'PY' import binascii data = binascii.unhexlify("735c455b72545f5d6f595e") key  = b"Ankh"                 # case matters out  = bytes([data[i]^key[i%len(key)] for i in range(len(data))]) print(out.decode()) PY`

**Flag:** `CTF{22.33:45.75}`
