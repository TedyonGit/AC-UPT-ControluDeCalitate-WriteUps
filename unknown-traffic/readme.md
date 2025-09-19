# unknown-traffic1

Can you find the hidden message in this pcap file?


# ICMP echo payloads → base64 → flag

Traffic hint said “echo request,” so I inspected **ICMP Echo Request** payloads. Each payload had a run of ASCII `'0'` (0x30) followed by a short useful chunk (4 bytes). Concatenating those 4-byte chunks in capture order produces a base64 blob that decodes to the flag.

## Steps

- Open the pcap (libpcap, little-endian) and treat packets as **raw IP** or Ethernet.

- Filter for IPv4 **ICMPv4 Echo Request** (type 8) and take the L4 payload (`icmp[8:]`).

- For each payload: strip leading ASCII `0x30` bytes, take the remaining 4 bytes (or last 4 if more).

- Concatenate chunks in capture order → `b64_blob`.

- Base64-decode `b64_blob` (pad to multiple of 4) and search for `ctf{...}`.

## Most important part of the script

Only the core extraction/decoding shown — omits pcap parser/robust heuristics for brevity.

```python
import struct, base64, re

# assume `pkts` is a list of raw IP packet bytes parsed from the pcap (capture order)
chunks = []
for pkt in pkts:
    # IPv4: extract L4 payload (simple, assumes no options mischief)
    ihl = (pkt[0] & 0x0F) * 4
    proto = pkt[9]
    if proto != 1: continue                  # not ICMP
    icmp = pkt[ihl:]
    if len(icmp) < 8 or icmp[0] != 8: continue  # not Echo Request
    payload = icmp[8:]
    rem = payload.lstrip(b'0')               # strip ASCII '0' (0x30)
    chunk = rem[:4] if len(rem) >= 4 else rem
    chunks.append(chunk)

b64_blob = b"".join(chunks)
# pad to multiple of 4 then decode
pad = (-len(b64_blob)) % 4
decoded = base64.b64decode(b64_blob + b"="*pad, validate=False)
m = re.search(rb'(ctf\{[^}]+\})', decoded, re.I)
print(m.group(1).decode() if m else decoded[:200])
```









# unknown-traffic2

A lot more traffic just showed up on the network. Can you figure out what it is?

# HTTP GET / ICMP chunks → base64 → PNG → flag

Capture contained numbered Base64 fragments sent as `?chunk=N&data=...` in HTTP GETs and `CHUNK_N:<b64>` in ICMP. Reassemble by chunk index, normalize Base64 (restore `+`, accept base64url, fix padding), join and decode — result is a PNG that contains the flag.

## Steps

- Parse pcap in capture order.

- Extract `chunk`/`data` from HTTP GET query (preserve `+`) and `CHUNK_{N}:<b64>` from ICMP payloads.

- For each index keep best candidate (longer string).

- Normalize each fragment: `space->+`, `-/_ -> +/ /`, strip junk, pad to multiple of 4.

- Sort by index, join, base64-decode → write file (if PNG, open/scan for flag).

```python
#!/usr/bin/env python3
# minimal_exfil.py
import re, base64, os
from scapy.all import rdpcap, TCP, ICMP, Raw
from urllib.parse import urlsplit, unquote

HTTP_GET_RE = re.compile(rb"GET\s+([^\s]+)\s+HTTP/1\.[01]")
ICMP_RE = re.compile(r"(?i)\bchunk[_-]?(\d+)\s*:\s*([A-Za-z0-9+/=_-]{4,})")

def norm(s):
    if isinstance(s, bytes): s = s.decode('latin1', 'ignore')
    s = s.strip().replace(' ', '+').replace('-', '+').replace('_', '/')
    s = re.sub(r'[^A-Za-z0-9+/=]', '', s)
    s += '=' * ((4 - len(s) % 4) % 4)
    return s

def extract(pcap):
    pkts = rdpcap(pcap)
    chunks = {}
    for p in pkts:
        if p.haslayer(TCP) and p.haslayer(Raw):
            raw = bytes(p[Raw].load)
            for m in HTTP_GET_RE.finditer(raw):
                uri = m.group(1).decode('ascii', 'ignore')
                q = urlsplit(uri).query
                if not q: continue
                kv = {}
                for pair in q.split('&'):
                    if '=' not in pair: continue
                    k,v = pair.split('=',1)
                    kv[k] = unquote(v)  # preserves '+' as '+'
                if 'chunk' in kv and 'data' in kv:
                    idx = int(kv['chunk']); val = kv['data']
                    if idx not in chunks or len(val) > len(chunks[idx]): chunks[idx] = val
        if p.haslayer(ICMP) and p.haslayer(Raw):
            txt = bytes(p[Raw].load).decode('latin1','ignore')
            for m in ICMP_RE.finditer(txt):
                idx = int(m.group(1)); val = m.group(2)
                if idx not in chunks or len(val) > len(chunks[idx]): chunks[idx] = val
    return chunks

def rebuild_and_write(chunks, out='recovered.bin'):
    if not chunks: raise SystemExit("no chunks")
    lo,hi = min(chunks), max(chunks)
    missing = [i for i in range(lo,hi+1) if i not in chunks]
    if missing: print("missing:", missing[:20])
    data = b''.join(base64.b64decode(norm(chunks[i])) for i in sorted(chunks))
    if data.startswith(b'\x89PNG\r\n\x1a\n'):
        out = os.path.splitext(out)[0]+'.png'
    open(out,'wb').write(data)
    print("wrote", out)
    return data

if __name__ == "__main__":
    chunks = extract("traffic.pcap")
    blob = rebuild_and_write(chunks, "recovered")
    # try to print inline flag if present
    import re
    m = re.search(rb'(CTF\{[^}]+\}|ctf\{[^}]+\})', blob, re.I)
    if m: print("FLAG:", m.group(1).decode())
    else: print("No flag in bytes — inspect recovered file (maybe QR/image).")
```

after we get an .png photo with qr code, that is flag
