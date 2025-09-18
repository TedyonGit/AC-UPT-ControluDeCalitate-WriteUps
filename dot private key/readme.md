# dot-private-key
We scanned the directories, found two end-points (not meant to be public) /dumb and /clear and after we played a bit we found the flag in there. (unintended way)

# neverending randomness
It was a state-recovery attack, we brute-forced the seed using this script

```python
import socket, ast, time, random

HOST = "ctf.ac.upt.ro"
PORT = 9466
WINDOW = 30      # seconds around receive time

def xor_bytes(a: bytes, b: bytes) -> bytes:
    return bytes(x ^ y for x, y in zip(a, b))

def get_snapshot():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(5)
    s.connect((HOST, PORT))
    data = s.recv(4096).decode().strip()
    ts_local = int(time.time())
    s.close()
    return ast.literal_eval(data), ts_local

def recover_flag(snap: dict, ts: int, window: int):
    ct = bytes.fromhex(snap["ciphertext_hex"])
    leak = snap["leak32"]
    pid = snap["pid"]
    ct_len = len(ct)

    for t in range(ts - window, ts + window + 1):
        seed = t ^ pid
        rng = random.Random(seed)

        # Generate keystream (same order as server)
        ks = bytearray()
        while len(ks) < ct_len:
            ks.append(rng.getrandbits(8))

        # Then generate leaks
        leaks = [rng.getrandbits(32) for _ in range(3)]
        if leaks == leak:
            flag = xor_bytes(ct, ks[:ct_len])
            return flag, seed, t
    return None, None, None

def main():
    attempt = 0
    while True:
        attempt += 1
        snap, ts = get_snapshot()
        print(f"[*] Attempt {attempt}: pid={snap['pid']} leaks={snap['leak32']}")

        flag, seed, t = recover_flag(snap, ts, WINDOW)
        if flag:
            print(f"[+] Seed found: {seed} (time={t}, pid={snap['pid']})")
            try:
                print(f"[+] FLAG: {flag.decode()}")
            except UnicodeDecodeError:
                print(f"[+] FLAG (bytes): {flag!r}")
            break
        else:
            print("[-] Failed this attempt, retrying...\n")

if __name__ == "__main__":
    main()
```
