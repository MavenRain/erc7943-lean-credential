# Sealed uRWA / ERC-7943 Lean specification: public commitment

This repository publishes a cryptographic commitment to a sealed (private) Lean 4
formal specification of ERC-7943 (uRWA), so that authorship and existence-by-date
are provable without disclosing the proofs.

The capability credential itself (the theorem statements and the green-build
attestation) is in `credential-note.md`.

What is committed (`COMMITMENT.txt`):
- `DIGEST` - SHA-256 over the per-file manifest of the sealed repository.
- `COMMIT` - the anchored git commit id of the sealed repository.

The commitment is signed by the author (`COMMITMENT.txt.sig`, verifiable against
`allowed_signers`) and anchored in Bitcoin via OpenTimestamps
(`COMMITMENT.txt.ots`).

- Anchor: Bitcoin block 953674 (2026-06-14 19:07:16 UTC).
- Signer: softwareengineerasaservant@isurvivable.cv, ed25519
  `SHA256:b6D6JxqxZm/brI4tH6N4X+yoEBjeDw4vrtesKmDmhfw`.

The digest is opaque and reveals nothing about the specification.  The full
artifact, the per-file manifest, and the proofs are disclosed only under a
mutual NDA.  Contact: softwareengineerasaservant@isurvivable.cv.

## Verify the commitment (no trust required)

```
# 1. Author signature over the commitment:
ssh-keygen -Y verify -f allowed_signers -I softwareengineerasaservant@isurvivable.cv \
  -n file -s COMMITMENT.txt.sig < COMMITMENT.txt

# 2. Bitcoin anchor (needs a Bitcoin node), or use https://opentimestamps.org:
ots verify COMMITMENT.txt.ots
```

## Reveal-and-verify (for a counterparty, under engagement)

Given the sealed repository checked out at the anchored `COMMIT`, and the files
published here:

```
git checkout <COMMIT>
git ls-files -z | LC_ALL=C sort -z | while IFS= read -r -d '' p; do
  printf '%s  %s\n' "$(git cat-file blob "HEAD:$p" | shasum -a 256 | awk '{print $1}')" "$p"
done > MANIFEST.regen.sha256
[ "$(shasum -a 256 MANIFEST.regen.sha256 | awk '{print $1}')" = "$(grep '^DIGEST=' COMMITMENT.txt | cut -d= -f2)" ] \
  && echo "MATCH: the revealed artifact is exactly the one committed here" \
  || echo "MISMATCH"
```

A single equality binds every tracked file of the sealed artifact: one changed
byte anywhere changes a blob hash, the manifest, and the digest.
