# Repetead RSA

We've been told that multiple RSA encryptions can strengthen the security. Not sure if it's alright though...
No source code required but we will provide one if there are no solves in the first 16 hours.


# Repeated RSA (3 moduli share primes) → gcd → factor → d’s → reverse-decrypt → send bytes

Service gives one ciphertext `C`, three moduli `n1,n2,n3` and `e=65537`.  
All three moduli share primes pairwise, i.e. `n1=p*q`, `n2=p*r`, `n3=q*r`.  
Compute the shared primes with `gcd`, recover private exponents, decrypt in reverse order, and **send the raw decrypted bytes** (no flag patterning).

## Steps

- Read `Encrypted Flag`, `n1`, `n2`, `n3`, `e` from the socket.

- Compute shared primes:  
  `p = gcd(n1,n2)`, `q = gcd(n1,n3)`, `r = gcd(n2,n3)`.

- Factor: `n1=p*q`, `n2=p*r`, `n3=q*r`.

- Compute `phi_i = (pi-1)*(qi-1)` and `d_i = e^{-1} mod phi_i` for i∈{1,2,3}.

- Assume encryption order was `n1 → n2 → n3`; decrypt in reverse:  
  `m = (((C^d3 mod n3)^d2 mod n2)^d1 mod n1)`.

- Convert `m` to bytes, strip newlines, send as a single line.

```python
#!/usr/bin/env python3
# repeated_rsa_client.py  — raw-bytes response, no pattern matching
import socket, sys, re, math

HOST = "ctf.ac.upt.ro"
PORT = 9850
TIMEOUT = 6.0

def recv_all(sock):
    sock.settimeout(TIMEOUT)
    buf = b""
    while True:
        try:
            chunk = sock.recv(4096)
            if not chunk: break
            buf += chunk
            if b"e:" in buf and b"\n" in buf.split(b"e:")[-1]: break
        except socket.timeout:
            break
    return buf

def parse_nums(txt):
    def grab(lbl):
        m = re.search(rf"{re.escape(lbl)}\s*([0-9]+)", txt)
        if not m: raise ValueError(f"missing {lbl}")
        return int(m.group(1))
    C  = grab("Encrypted Flag:")
    n1 = grab("n1:")
    n2 = grab("n2:")
    n3 = grab("n3:")
    e  = grab("e:")
    return C,n1,n2,n3,e

def modinv(a, m):
    try:
        return pow(a, -1, m)
    except ValueError:
        # extended Euclid fallback
        def egcd(x,y):
            if y==0: return (x,1,0)
            g,a1,b1 = egcd(y, x%y)
            return (g, b1, a1 - (x//y)*b1)
        g,x,_ = egcd(a,m)
        if g != 1: raise
        return x % m

def i2b(n:int)->bytes:
    return n.to_bytes((n.bit_length()+7)//8, "big") if n else b"\x00"

def main():
    host = HOST if len(sys.argv)<2 else sys.argv[1]
    port = PORT if len(sys.argv)<3 else int(sys.argv[2])

    with socket.create_connection((host, port), timeout=TIMEOUT) as s:
        banner = recv_all(s).decode(errors="ignore")
        C,n1,n2,n3,e = parse_nums(banner)

        # shared primes
        p = math.gcd(n1,n2)
        q = math.gcd(n1,n3)
        r = math.gcd(n2,n3)
        assert p>1 and q>1 and r>1, "expected shared primes"

        # factor each modulus as (pi, qi)
        assert n1 % p == 0 and n1 % q == 0
        assert n2 % p == 0 and n2 % r == 0
        assert n3 % q == 0 and n3 % r == 0
        p1,q1 = p, q
        p2,q2 = p, r
        p3,q3 = q, r

        phi1 = (p1-1)*(q1-1)
        phi2 = (p2-1)*(q2-1)
        phi3 = (p3-1)*(q3-1)
        d1 = modinv(e, phi1)
        d2 = modinv(e, phi2)
        d3 = modinv(e, phi3)

        # assume enc order: n1 -> n2 -> n3 ; decrypt: n3, then n2, then n1
        m = pow(C, d3, n3)
        m = pow(m, d2, n2)
        m = pow(m, d1, n1)
        out = i2b(m).replace(b"\r", b"").replace(b"\n", b"")

        s.sendall(out + b"\n")
        resp = recv_all(s).decode(errors="ignore")
        print(resp)
        # local echo of what we sent
        print("[sent bytes hex]:", out.hex())

if __name__ == "__main__":
    main()
```
