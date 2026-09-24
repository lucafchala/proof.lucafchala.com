# CLAUDE.md — proof.lucafchala.com

A PGP-signed identity statement. Static HTML + CSS, **no JavaScript at all** (CSP `script-src 'none'`).

- **Never edit anything inside `<pre class="signed">`** — not whitespace, not indentation, not line endings. The block is the exact clearsigned text; any change breaks the signature. CI runs `gpg --verify` on it and fails otherwise. Restyle around it only.
- **Keep the `<!--email_off-->…<!--/email_off-->` wrapper.** Without it, Cloudflare Email Obfuscation rewrites the addresses, and there is no script to decode them.
- **`proof.txt.asc` and `proof-of-ownership.txt` must stay identical to the signed block.** status.lucafchala.com checks `proof-of-ownership.txt` for `Luca Ferriani Chala`.
- **`pgp.asc`** is a vendored copy of keys.lucafchala.com/pgp.asc, used by CI.
- **Changing the statement requires re-signing it** (see README → "Changing the statement"). An assistant can't do that; ask the owner.
- **Styles** live in `proof.css` (style-src `'self'`, no inline styles). The theme comes from `prefers-color-scheme`.
