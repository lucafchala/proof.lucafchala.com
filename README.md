# proof.lucafchala.com

> A PGP-signed statement of the domains, addresses and accounts that belong to Luca Ferriani Chala, and how to verify it.

**Live:** [proof.lucafchala.com](https://proof.lucafchala.com) · **Stack:** static HTML + CSS, **no JavaScript** · **Host:** Cloudflare Pages

Part of the [lucafchala.com ecosystem](https://github.com/lucafchala/lucafchala.com#the-ecosystem). Design system: [hub README](https://github.com/lucafchala/lucafchala.com#design-system).

---

## What it is

The statement is a clearsigned PGP message, signed on 2026‑06‑08 by key `48E7 3F6F A287 1E7B 86EF  EA64 8EC4 329A 369B 7B33`. The page shows it verbatim, with a "how to verify" section and download links.

## Verify it

```bash
curl -fsSL https://keys.lucafchala.com/pgp.asc | gpg --import
curl -fsSL https://proof.lucafchala.com/proof.txt.asc | gpg --verify
# → Good signature from "Luca Ferriani Chala <lfchala4@gmail.com>"
#   Primary key fingerprint: 48E7 3F6F A287 1E7B 86EF  EA64 8EC4 329A 369B 7B33
```

## Files

| File | What |
|---|---|
| `index.html` | The page. The statement sits inside `<pre class="signed">` **byte-for-byte as signed**, wrapped in `<!--email_off-->…<!--/email_off-->` |
| `proof.txt.asc` | The same clearsigned statement as plain text |
| `proof-of-ownership.txt` | Also the current signed statement (kept at this path because the status monitor and older links use it) |
| `pgp.asc` | Vendored copy of the public key (identical to keys.lucafchala.com/pgp.asc), used by CI to verify |
| `proof.css` | Styles; theme follows the OS (`prefers-color-scheme`), since there is no JS |
| `icon.svg`, `fonts/` | Icon and self-hosted fonts |
| `_headers` | CSP `default-src 'none'; script-src 'none'; style-src 'self'; font-src 'self'; img-src 'self'`, HSTS, COOP/CORP, `text/plain` + CORS for the `.asc`/`.txt` files, `Link` hints |
| `sitemap.xml`, `robots.txt`, `.well-known/security.txt` | Discovery |

## Why `<!--email_off-->`

The Cloudflare zone has Email Obfuscation on. It rewrites every address in the HTML to `[email protected]` and relies on a decoder script to restore it. This site allows **no** scripts, so visitors saw `[email protected]` inside the signed text, which then no longer matched the signature. The `email_off` comments tell Cloudflare to leave that block alone.

## Changing the statement

Any change inside the `<pre>` — even re-indenting — invalidates the signature. To change the text:

1. Write the new statement and clearsign it: `gpg --clearsign --local-user 48E73F6FA2871E7B86EFEA648EC4329A369B7B33 statement.txt`.
2. Put the output in `proof.txt.asc` and `proof-of-ownership.txt`.
3. Paste it into the `<pre class="signed">` block of `index.html`, HTML-escaping `&`, `<` and `>` (the current text has none).
4. Update the "Signed" date on the page and in `sitemap.xml`.

CI refuses to merge anything whose signature doesn't verify.

## CI (`.github/workflows/checks.yml`)

- `_headers` present; **no `<script>` or `on*=` anywhere**.
- `gpg --verify` with the vendored key on: the text extracted from the page, `proof.txt.asc`, and `proof-of-ownership.txt`. The job requires a `VALIDSIG` from `48E73F6F…7B33`.
- The signed `<pre>` is wrapped in `email_off`.
- `proof-of-ownership.txt` contains `Luca Ferriani Chala` (the status monitor's marker).

## Status

**In production.**
