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

## Album launch notice (from fotos)

When fotos.lucafchala.com launches an album with **"Destaque máximo no site"** on, this page shows a blue card at the top of its column ("TURMA 2026 · COLÉGIO SANTA CRUZ **Entre Nós** · o álbum está no ar — Ver o álbum →"). Until then nothing changes.

- **Server-side, no script on the page.** This page ships **no JavaScript at all** (CSP `script-src 'none'`), and that stays true: the strip is plain HTML + CSS. `functions/_middleware.js` (only for `/`, see `_routes.json`) asks `https://fotos.lucafchala.com/api/destaque`, and if there's a launched album it inserts the strip right after `<body>` and `<link rel="stylesheet" href="/destaque.css">` in `<head>`. The CSP is unchanged.
- **fotos' answer is foreign data:** only the title, the label and an `https:` link to a known album host (`HOSTS_DE_ALBUM`: `fotos.lucafchala.com`, `entrenos2026.com`) pass, all HTML-escaped. fotos down, slow (2 s) or answering junk → the page is served untouched.
- **Cost:** one call to fotos per 5 min per Cloudflare location (edge cache; 1 min after a failure), and one Pages Functions invocation per visit to `/`.
- **ETag:** the function strips `If-None-Match`/`If-Modified-Since` and the response's `ETag`/`Last-Modified` on `/`, or a browser would keep revalidating its old copy without the strip.
- **Closing** (×) is a CSS checkbox: it hides the strip for that page view; it comes back on the next visit (there's no script to remember it). Hidden when printing.
- **Files:** `functions/_middleware.js`, `functions/_lib/destaque-servidor.js` (the logic — an **identical copy** lives in keys, pays and proof, and inline in rg's `_worker.js`; change all of them together), `destaque.css`, `_routes.json`, `tests/destaque.test.mjs`. The full picture across all sites is in fotos' `docs/ALBUM.md` ("Aviso nos outros sites").
- **A new album domain in fotos** (`DOMINIOS_DE_ALBUM`) must be added to `HOSTS_DE_ALBUM`, or the strip silently stays off for it.
