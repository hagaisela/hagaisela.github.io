# hagaisela.github.io — ToneSmith App Links trust anchor

This GitHub user-site repository exists for ONE purpose: serving
[`/.well-known/assetlinks.json`](.well-known/assetlinks.json) at the **domain root**, so
Android can verify the App Link claim of the ToneSmith app (package `tonesmith.app`) for
the link form `https://hagaisela.github.io/tone?spec=…`.

Android's verifier reads the statements file ONLY at the domain root — this is why the
anchor lives in the user-site repo (published at `/`) rather than a project repo's Pages
(which would serve it under `/tonesmith/`, a path prefix verification never reads).

## Content boundary

This repo deliberately publishes nothing beyond the app's package name, the signing
certificate fingerprint(s), and one static index paragraph: no keys, no endpoints, no
secrets, no personal data beyond the GitHub account itself. Keep it that way.

## `.nojekyll` must NEVER be deleted

GitHub Pages builds sites through Jekyll by default, and Jekyll **skips dot-directories**.
Without the `.nojekyll` file at the repo root, `/.well-known/assetlinks.json` would 404
even though it is committed — silently breaking App Link verification. If the site ever
404s on the statements file, check this file first.

## Release-key law: APPEND, never replace

`sha256_cert_host_fingerprints` currently holds the debug keystore's SHA-256 fingerprint
(the key every rig debug build signs with). When a release keystore is created, add its
fingerprint as a **SECOND entry in the array** — never replace or remove the debug one:

```json
"sha256_cert_host_fingerprints": [
  "2A:DE:...debug...",
  "AA:BB:...release..."
]
```

Android accepts multiple fingerprints for exactly this migration; deleting the debug entry
would un-verify every debug build the moment the push publishes.

Verify the live file after any change:
`curl -s https://hagaisela.github.io/.well-known/assetlinks.json`

## /tone — the browser bridge

`/tone` is a fallback page for devices where the App Link is not (yet) verified: it
forwards the `spec` parameter of `https://hagaisela.github.io/tone?spec=…` to the app's
`tonesmith://tone` custom scheme via a redirect plus a manual button. Once Android
verifies the App Link, the system opens the app directly and this page is rarely seen.
