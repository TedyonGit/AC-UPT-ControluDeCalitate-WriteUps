# Theme-Generator

Just a simple theme generator app. Nothing fancy.

## Theme Forge – Prototype Pollution → Admin

### Steps

- Service gives a guest login and endpoints:
  
  - `/api/preset/upload` → merges uploaded JSON with `deepMerge`.
  
  - `/admin/flag` → only for admins (`req.user.isAdmin`).

- `deepMerge` recurses into objects and writes keys blindly.  
  WAF blocks **top-level** `__proto__`, `constructor`, `prototype`, but **nested** keys pass.

- Pollution trick:  
  Upload preset JSON:
  
  ```json
  {
    "safe": {
      "constructor": {
        "prototype": {
          "isAdmin": true
        }
      }
    }
  }
  ```
  
  → Sets `Object.prototype.isAdmin = true`.

- Log in as guest (`guest/guest`), upload payload, then visit `/admin/flag`.

- Response returns the raw flag.
