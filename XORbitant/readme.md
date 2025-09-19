# XORbitant

Let us master the primordial cryptoanalysis technique. May the first ring of crypto techniques be with you.The flag is key.

- On this challenge we had a ``.bin`` file that was encrypted with the ``enc.py``.

- In order to recover the flag we needed to find the key, so we came up with a python script that fix that for us.

```python
#!/usr/bin/env python3
import hashlib
from itertools import cycle

def recover_xor_key(ciphertext_path: str):
    """
    Recover XOR key by analyzing the encrypted file.
    This assumes the plaintext contains readable English text.
    """
    
    with open(ciphertext_path, "rb") as f:
        ciphertext = f.read()
    
    print(f"Ciphertext length: {len(ciphertext)} bytes")
    print(f"First 32 bytes (hex): {ciphertext[:32].hex()}")
    
    # Method 1: Try common key lengths and look for readable patterns
    # CTF flags often start with "CTF{" so let's try that
    flag_start = b"CTF{"
    
    print("\n=== Trying known plaintext attack ===")
    
    # If we assume the plaintext starts with common text patterns
    common_starts = [
        b"CTF{",
        b"flag{",
        b"The ",
        b"This ",
        b"Hello",
        b"Welcome",
        b"Congratulations",
        b"Well done",
        b"Good job"
    ]
    
    for start in common_starts:
        if len(start) <= len(ciphertext):
            # XOR the start of ciphertext with assumed plaintext
            potential_key_start = bytes([c ^ p for c, p in zip(ciphertext, start)])
            print(f"If plaintext starts with '{start.decode('utf-8', errors='ignore')}': key starts with '{potential_key_start.decode('utf-8', errors='ignore')}'")
    
    # Method 2: Statistical analysis - look for repeating patterns
    print("\n=== Statistical Analysis ===")
    
    # Try different key lengths, focusing on 69 chars (CTF{64-char-sha256})
    key_lengths_to_try = [69, 68, 70, 67, 71]  # Focus on expected length first
    
    for key_len in key_lengths_to_try:
        key_candidate = bytearray(key_len)
        
        # For each position in the key, find the most likely byte
        for pos in range(key_len):
            byte_counts = {}
            
            # Collect all bytes that were XORed with this key position
            for i in range(pos, len(ciphertext), key_len):
                cipher_byte = ciphertext[i]
                
                # Try XORing with common English characters
                for common_char in b' etaoinshrdlcumwfgypbvkjxqz':
                    key_byte = cipher_byte ^ common_char
                    byte_counts[key_byte] = byte_counts.get(key_byte, 0) + 1
            
            if byte_counts:
                # Pick the most frequent candidate
                key_candidate[pos] = max(byte_counts, key=byte_counts.get)
        
        # Test if this key produces readable text
        try:
            key_str = key_candidate.decode('utf-8')
            if key_str.startswith('CTF{') and key_str.endswith('}'):
                print(f"\n*** POTENTIAL FLAG FOUND ***")
                print(f"Key length: {key_len}")
                print(f"Key: {key_str}")
                
                # Verify by decrypting
                decrypted = decrypt_with_key(ciphertext, key_candidate)
                print(f"Decrypted text preview: {decrypted[:100].decode('utf-8', errors='ignore')}")
                
                # Calculate SHA256
                sha256_hash = hashlib.sha256(key_str.encode()).hexdigest()
                print(f"SHA256: {sha256_hash}")
                print(f"Final flag format: CTF{{{sha256_hash}}}")
                return key_str
                
        except UnicodeDecodeError:
            continue
    
    print("No obvious flag found with statistical analysis.")
    return None

def decrypt_with_key(ciphertext: bytes, key: bytes) -> bytes:
    """Decrypt ciphertext using the given key"""
    key_cycle = cycle(key)
    return bytes([c ^ next(key_cycle) for c in ciphertext])

def targeted_recovery(ciphertext_path: str):
    """
    Targeted recovery knowing the flag format is CTF{sha256}
    Key length should be exactly 69 characters
    """
    with open(ciphertext_path, "rb") as f:
        ciphertext = f.read()
    
    print(f"=== Targeted Recovery (CTF{{sha256}} format) ===")
    key_len = 69
    
    # We know the flag starts with "CTF{" and ends with "}"
    known_prefix = b"CTF{"
    known_suffix = b"}"
    
    # Try known plaintext attack with common CTF beginnings
    common_plaintexts = [
        b"The flag is CTF{",
        b"Congratulations! CTF{", 
        b"Well done! CTF{",
        b"Flag: CTF{",
        b"Your flag: CTF{",
        b"CTF{",
        b"Here is your flag: CTF{",
        b"Success! The flag is CTF{"
    ]
    
    for plaintext_start in common_plaintexts:
        if len(plaintext_start) <= len(ciphertext):
            # Calculate what the key would be
            potential_key_bytes = []
            for i in range(min(len(plaintext_start), len(ciphertext))):
                key_byte = ciphertext[i] ^ plaintext_start[i]
                potential_key_bytes.append(key_byte)
            
            # Extend the pattern to full key length
            if len(potential_key_bytes) >= 4:  # At least "CTF{"
                # Create full key by repeating the pattern
                full_key = bytearray(key_len)
                for i in range(key_len):
                    full_key[i] = potential_key_bytes[i % len(potential_key_bytes)]
                
                # Check if this creates a valid CTF{sha256} pattern
                try:
                    key_str = full_key.decode('utf-8')
                    if (key_str.startswith('CTF{') and 
                        key_str.endswith('}') and 
                        len(key_str) == 69 and
                        all(c in '0123456789abcdefABCDEF' for c in key_str[4:68])):
                        
                        print(f"\n*** FLAG FOUND! ***")
                        print(f"Assumed plaintext start: '{plaintext_start.decode()}'")
                        print(f"Recovered flag: {key_str}")
                        
                        # Verify by decrypting
                        decrypted = decrypt_with_key(ciphertext, full_key)
                        print(f"Decrypted preview: {decrypted[:100].decode('utf-8', errors='ignore')}")
                        
                        return key_str
                        
                except UnicodeDecodeError:
                    continue
    
    # If known plaintext didn't work, try statistical analysis with known structure
    print(f"\nTrying statistical analysis with 69-character key...")
    key_candidate = bytearray(key_len)
    
    # We know positions 0-3 should be "CTF{" and position 68 should be "}"
    # Use this to constrain our search
    target_chars = {0: ord('C'), 1: ord('T'), 2: ord('F'), 3: ord('{'), 68: ord('}')}
    
    for pos in range(key_len):
        if pos in target_chars:
            # For known positions, calculate what the key byte should be
            # by trying common plaintext characters
            candidates = []
            for plaintext_char in b' etaoinshrdlcumwfgypbvkjxqz':
                key_byte = target_chars[pos] ^ plaintext_char
                if 32 <= key_byte <= 126:  # Printable ASCII
                    candidates.append(key_byte)
            
            if candidates:
                key_candidate[pos] = candidates[0]  # Take first valid candidate
        else:
            # For SHA256 positions (4-67), look for hex characters
            byte_counts = {}
            for i in range(pos, len(ciphertext), key_len):
                cipher_byte = ciphertext[i]
                
                # Try XORing with hex characters
                for hex_char in b'0123456789abcdef':
                    key_byte = cipher_byte ^ hex_char
                    if 32 <= key_byte <= 126:
                        byte_counts[key_byte] = byte_counts.get(key_byte, 0) + 1
            
            if byte_counts:
                key_candidate[pos] = max(byte_counts, key=byte_counts.get)
    
    # Check if we got a valid flag
    try:
        key_str = key_candidate.decode('utf-8')
        if (key_str.startswith('CTF{') and 
            key_str.endswith('}') and 
            len(key_str) == 69):
            
            print(f"\n*** POTENTIAL FLAG (Statistical) ***")
            print(f"Flag: {key_str}")
            
            decrypted = decrypt_with_key(ciphertext, key_candidate)
            print(f"Decrypted preview: {decrypted[:100].decode('utf-8', errors='ignore')}")
            
            return key_str
            
    except UnicodeDecodeError:
        pass
    
    return None
    """Try brute force for very short keys"""
    with open(ciphertext_path, "rb") as f:
        ciphertext = f.read()
    
    print("\n=== Brute force for short keys ===")
    
    # Try single character keys (unlikely but possible)
    for key_byte in range(32, 127):  # Printable ASCII
        key = bytes([key_byte])
        decrypted = decrypt_with_key(ciphertext, key)
        
        # Check if result looks like text
        try:
            text = decrypted.decode('utf-8')
            if 'CTF' in text or 'flag' in text:
                print(f"Single byte key {key_byte} ({chr(key_byte)}): {text[:50]}...")
        except:
            continue

def manual_analysis(ciphertext_path: str):
    """Manual analysis helper"""
    with open(ciphertext_path, "rb") as f:
        ciphertext = f.read()
    
    print(f"\n=== Manual Analysis Helper ===")
    print(f"File size: {len(ciphertext)} bytes")
    print(f"First 64 bytes (hex):")
    for i in range(0, min(64, len(ciphertext)), 16):
        chunk = ciphertext[i:i+16]
        hex_str = ' '.join(f'{b:02x}' for b in chunk)
        ascii_str = ''.join(chr(b) if 32 <= b <= 126 else '.' for b in chunk)
        print(f"{i:04x}: {hex_str:<48} {ascii_str}")

if __name__ == "__main__":
    ciphertext_file = "out.bin"
    
    print("XOR Key Recovery Tool for CTF Challenge")
    print("Target: CTF{sha256} format (69 characters)")
    print("=" * 50)
    
    # Try targeted recovery first
    recovered_key = targeted_recovery(ciphertext_file)
    
    if not recovered_key:
        # Fallback to general recovery
        print("\nTargeted recovery failed, trying general approach...")
        recovered_key = recover_xor_key(ciphertext_file)
    
    if not recovered_key:
        manual_analysis(ciphertext_file)
        
        print("\n" + "=" * 50)
        print("MANUAL STEPS:")
        print("1. The flag should be exactly 69 characters: CTF{64-hex-chars}")
        print("2. Try XORing start of ciphertext with 'CTF{' to get key start")
        print("3. The key repeats every 69 bytes")
        print("4. Middle should be hex characters (0-9, a-f)")
        print("5. Should end with '}'")
    
    if recovered_key:
        print(f"\n🎉 SUCCESS! Recovered flag: {recovered_key}")
        print(f"This IS the final flag for submission!")
```
