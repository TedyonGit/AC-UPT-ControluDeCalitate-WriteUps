# disco_rave

This is an slighlty harder version of disco_dance because it takes last 10 messages+timestamps from 2 channesl, using the same method to see what channels were used I tried the same script but hard-coding it was much harder so I automated it. I used my own discord token to get my fetch the messages and timestamps.

```python
import socket
import ast
import base64
import requests
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util.Padding import unpad

# === CONFIG ===
HOST = "ctf.ac.upt.ro"
PORT = 9135
DISCORD_TOKEN = "YOU_AINT_SEEING_THIS" 

CHANNELS = [
    "1416908413375479891",
    "1417154025371209852",
]

# === FUNCTIONS ===

def fetch_seed():
    """Fetch messages from both channels and rebuild the seed"""
    headers = {
        "Authorization": DISCORD_TOKEN,
        "User-Agent": "Mozilla/5.0",  # Discord sometimes checks this
    }
    all_data = []

    for channel_id in CHANNELS:
        url = f"https://discord.com/api/v10/channels/{channel_id}/messages?limit=10"
        r = requests.get(url, headers=headers)
        r.raise_for_status()
        messages = r.json()

        for msg in messages:
            content = msg.get("content", "")
            timestamp = msg.get("timestamp", "")
            all_data.append(f"{content}{timestamp}")

    seed = "".join(all_data).encode("utf-8")
    return seed

def derive_key(seed: bytes):
    digest = SHA256.new()
    digest.update(seed)
    return digest.digest()

def get_ciphertext():
    """Connect to challenge server and get ciphertext dict"""
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    data = s.recv(4096).decode().strip()
    s.close()
    out = ast.literal_eval(data)
    return out["encrypted"]

def decrypt_flag(encrypted_b64, key):
    raw = base64.b64decode(encrypted_b64)
    iv, ciphertext = raw[:16], raw[16:]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)
    return decrypted.decode()

# === MAIN EXPLOIT ===
if __name__ == "__main__":
    print("[+] Fetching ciphertext from challenge...")
    encrypted_b64 = get_ciphertext()

    print("[+] Fetching Discord messages...")
    seed = fetch_seed()
    key = derive_key(seed)

    print("[+] Decrypting...")
    flag = decrypt_flag(encrypted_b64, key)

    print("\nFLAG:", flag)
```
