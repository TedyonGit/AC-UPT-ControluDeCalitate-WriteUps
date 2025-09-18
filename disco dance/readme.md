# disco_dance (first blood)
This used an ASE encryption and used a seed resulted from fetching and concatenating the last 5 messages from a discord channel using a proxy. The channel id was visible in the proxy url so by putting it like this <#channel-id> I could see exactly what channel was used. I wrote 5 messages on that channel and hard-coded it as the seed for my script and got the flag.


```python
import socket
import ast
import base64
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util.Padding import unpad

# Challenge server details
HOST = "ctf.ac.upt.ro"
PORT = 9388

# Step 1: Connect to server and receive ciphertext
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))

data = s.recv(4096).decode().strip()
s.close()

# The server sends a Python dict as a string, so we can parse it
out = ast.literal_eval(data)
encrypted_b64 = out["encrypted"]

print("[+] Received ciphertext:", encrypted_b64)

# Step 2: Compute the seed
seed = b"spamspamspamspamspam"

# Step 3: Derive AES key
digest = SHA256.new()
digest.update(seed)
aes_key = digest.digest()

# Step 4: Decode and decrypt
raw = base64.b64decode(encrypted_b64)
iv, ciphertext = raw[:16], raw[16:]

cipher = AES.new(aes_key, AES.MODE_CBC, iv)
decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)

print("[+] FLAG:", decrypted.decode())
```
