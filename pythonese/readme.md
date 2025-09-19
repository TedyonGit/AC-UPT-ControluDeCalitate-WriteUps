# pythonese

Wait, this language compiles ?...


# PyLingual `.pyc` decompile → flag

Used **PyLingual** to decompile the `.pyc` → got noisy code but constants + permutation visible.

## Steps

- Extract tables `A..J` and `P` from decompile.

- Transform: `b = (((v >> 1) - k2) ^ k1) & 0xFF`.

- Reverse chunks, permute with `P`, join `a + u + j`.

- `k2 = 45` known → brute-force `k1` 0–255.

## Script

```python
A = (382,356,392,430)
B = (442,544,456,552,544,538,540,446)
# ... other tables ...
P = (3,6,1,7,0,5,2,4)

def fvdy(arr,k1,k2):
    return bytes((((v>>1)-k2)^k1)&0xFF for v in arr)

def rebuild(k1,k2):
    parts=[fvdy(x,k1,k2) for x in [B,C,D,E,F,G,H,I]]
    parts=[s[::-1] for s in parts]
    R=[b'' for _ in range(8)]
    for i,p in enumerate(P): R[p]=parts[i]
    return fvdy(A,k1,k2)+b''.join(s[::-1] for s in R)+fvdy(J,k1,k2)

for k1 in range(256):
    out=rebuild(k1,45)
    try:
        s=out.decode()
    except: continue
    if s.startswith("CTF{"): print(s);break
```

## Result

Flag → `CTF{2944cec0c0f401a5fa538933a2f6210c279fbfc8548ca8ab912b493d03d2f5bf}`
