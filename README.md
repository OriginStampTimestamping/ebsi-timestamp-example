# EBSI Timestamp Example

A single-file, dependency-light web app to **create** and **verify** timestamps on the
[EBSI](https://hub.ebsi.eu) pilot ledger. Drop a file, hash it locally with SHA-256, and either
anchor the hash on-chain or check whether it was anchored before.

**Live demo:** https://originstamptimestamping.github.io/ebsi-timestamp-example/

Built by [OriginStamp](https://originstamp.com).

---

## What it does

The app has two tabs:

- **Create timestamp** — hash a file, sign the transaction with your `ES256K` key, and submit it to
  the EBSI Timestamp API. Requires an onboarded `did:ebsi` Legal Entity and a short-lived access token.
- **Verify timestamp** — hash a file and look up whether that exact hash exists on the ledger.
  Public read, **no keys or token required**.

## How it works

Everything runs in the browser. No backend, no build step.

| Step | Technique |
|------|-----------|
| File hashing (SHA-256) | native Web Crypto (`crypto.subtle.digest`) |
| Access token | empty Verifiable Presentation signed with `ES256` (P-256) via native Web Crypto, exchanged at the EBSI Authorisation API (`grant_type=vp_token`) |
| Transaction signing (secp256k1) | [ethers.js](https://github.com/ethers-io/ethers.js) v6, pinned from cdnjs |
| Timestamp write | `timestampHashes` → sign → `sendSignedTransaction` (Timestamp API v4 JSON-RPC) |
| Verify | derive `timestampId = 'u' + base64url( 0x1220 ‖ sha256(hashBytes) )`, then `GET /timestamp/v4/timestamps/{id}` |

`hashAlgorithmIds: 0` = SHA-256 (1 = SHA-384, 2 = SHA-512).

## Prerequisites (for creating timestamps)

1. A `did:ebsi` **Legal Entity** onboarded on the pilot network (has a `VerifiableAuthorisationToOnboard`
   and is registered on the DID Registry).
2. Two keys bound to that DID:
   - **ES256K** (secp256k1) — signs the on-chain transaction.
   - **ES256** (P-256, JWK) — signs the presentation used to fetch the access token.
3. An access token with scope `timestamp_write`. Use the built-in **Get token** button (it signs an empty
   presentation with your ES256 key), or paste one obtained via the [EBSI CLI](https://hub.ebsi.eu/tools/cli).
   Tokens are valid for ~2 hours.

Verifying needs none of the above.

## Usage

1. Open the [live demo](https://originstamptimestamping.github.io/ebsi-timestamp-example/) (or `index.html` locally — see below).
2. **Verify:** switch to the *Verify timestamp* tab, drop a file, click *Verify on EBSI*.
3. **Create:** open *Settings*, enter your DID, ES256K key and ES256 JWK, click *Get token*, save.
   Then on the *Create timestamp* tab, drop a file and click *Send timestamp*.

## Security

- **Keys and tokens live in memory only.** Nothing is stored, persisted, or sent anywhere except directly
  to the EBSI API. There is no analytics, no backend, no `localStorage`.
- **Never commit key material.** A `.gitignore` blocks common key/token file patterns. Enter secrets in the
  UI at runtime — do not hardcode them into `index.html`.
- This targets the **pilot (test) network** `api-pilot.ebsi.eu`. It is not production.
- You can verify the pinned ethers.js bundle and add Subresource Integrity — see the comment at the top of `index.html`.

## Running locally

Because the app calls the EBSI API, open it over HTTP rather than `file://` to avoid browser CORS issues:

```bash
python3 -m http.server
# then open http://localhost:8000
```

Hashing works offline regardless. Whether the API calls succeed cross-origin depends on EBSI's CORS policy;
each request also prints an equivalent `curl` command in the log as a fallback.

## Hosting on GitHub Pages

`index.html` sits at the repository root, so Pages can serve it directly. Two options:

**Option A — GitHub Actions (included).** This repo ships `.github/workflows/deploy.yml`. In
**Settings → Pages → Build and deployment → Source**, choose **GitHub Actions**. Every push to `main` deploys.

**Option B — deploy from a branch (no workflow needed).** In **Settings → Pages → Source**, choose
**Deploy from a branch**, then branch `main` / folder `/ (root)`. Delete the workflow file if you use this.

The site will be published at `https://originstamptimestamping.github.io/ebsi-timestamp-example/`.

## References

- [EBSI Timestamp API v4](https://hub.ebsi.eu/apis/pilot/timestamp/v4)
- [EBSI Authorisation API v4](https://hub.ebsi.eu/apis/pilot/authorisation/v4)
- [Timestamp docs](https://hub.ebsi.eu/api/timestamp/)

## License

No license is set yet. Add one (e.g. MIT) before relying on this as reusable open source.
