# ERC-7943 (uRWA) Formal Specification: Public Capability Credential

> STATUS: anchored in Bitcoin block 953674 (2026-06-14).  Ready to publish once
> the Author display name below is set.

Author: Onyeka Obi (MavenRain), independent formal-verification engineer.  Contact: softwareengineerasaservant@isurvivable.cv.  Date: 2026-06-14.

## What this is

A public, method-free credential.  It shows that a sealed Lean 4 formal specification of ERC-7943 (the uRWA real-world-asset token standard, in its Final form) exists, builds green, and mechanically enforces the standard's core safety invariants as a reachable-state global property.  It publishes only theorem statements, a green-build attestation, and a cryptographic commitment to the sealed artifact, so that priority is provable without disclosure.  The proof bodies, the invariant-decomposition layer, and the bytecode-semantics bridge are held privately and are available under a paid scoping engagement covered by a mutual NDA.  Predicate names below are the model's own; the underlying definitions are not disclosed here.

## Invariants discharged (statements only; proofs in the sealed artifact)

`FreezeInv s` abbreviates "no account's frozen amount exceeds its balance".

```lean
-- 1. Freeze safety (preservation): any user or forced transfer preserves FreezeInv.
theorem freeze_safety {s s' : State} {t : Transfer}
    (hinv : FreezeInv s) (hstep : Step s t s') : FreezeInv s'

-- 2. Allowlist gating: a non-forced transfer requires the sender on the
--    send-list and the receiver on the receive-list (the standard's
--    canSend / canReceive, modeled separately).
theorem allowlist_gating {s s' : State} {t : Transfer}
    (hstep : Step s t s') (hk : t.kind = TransferKind.user) :
    s.allowedSend t.sender = true ∧ s.allowedReceive t.receiver = true

-- 3. Forced-transfer authority: only the enforcement role may force a transfer.
theorem forced_authority {s s' : State} {t : Transfer}
    (hstep : Step s t s') (hk : t.kind = TransferKind.forced) :
    s.role t.actor Role.enforcer = true

-- 4. Batch freeze safety (FLAGSHIP): a user batch transfer preserves FreezeInv,
--    even when the batch repeats a token id.  This is the property the ERC-1155
--    reference violates (ethereum/ERCs #1814).
theorem batch_freeze_safety {s s' : State} {b : BatchTransfer}
    (hinv : FreezeInv s) (hstep : StepBatch s b s') : FreezeInv s'

-- 5. Global invariant: every state reachable from genesis by any modeled
--    operation (transfer, batch, mint, burn, freeze-issuance, allowlist/role
--    admin) satisfies freeze safety.  Established at genesis and closed under
--    every operation, so it is a system-wide guarantee, not a per-call one.
theorem reachable_freeze {s : State} (h : Reachable s) : FreezeInv s
```

One line each: (1) ordinary and enforcement transfers never push a holder below its compliance freeze; (2) ordinary holders transact only with cleared parties; (3) the only override of the freeze or allowlist runs through the explicit enforcement role; (4) the freeze holds against a whole batch, not each entry in isolation; (5) freeze safety holds in every reachable state of the full lifecycle, including the freeze-issuance operation that raises the freeze.

## Bug-catch

Statement 4 is the property the public ERC-1155 reference violates.  A batch that repeats a token id can drive a holder's post-state below its frozen amount, so the reference admits a transition that statement 4 forbids; a specification-conformant implementation cannot.  Disclosure: https://github.com/ethereum/ERCs/issues/1814.  The sealed artifact's test target proves this separation directly: the buggy per-entry guard provably breaks `FreezeInv` on the duplicate-id batch, while the spec's cumulative guard has no such transition.

## Green-build attestation

```
Artifact:           ERC-7943 / uRWA Lean 4 formal specification (sealed)
Lean toolchain:     leanprover/lean4:v4.30.0
Dependencies:       none (core Lean only)
Build status:       GREEN (lake build, 0 errors, 0 warnings)
Core theorems:      10  (4 interface invariants, 5 lifecycle-closure lemmas,
                        1 reachable-state global invariant)
Incomplete proofs:  0  (no `sorry`)
Axioms:             {propext, Quot.sound} only (no Classical.choice),
                        checked via #print axioms on every theorem
Test target:        inhabitation witnesses + a machine-checked proof that the
                        buggy per-entry guard violates freeze safety while the
                        spec forbids it (axiom-free)
```

Scope (v0.1): transfer-and-lifecycle freeze safety as a reachable-state invariant.  Supply conservation and operator-approval semantics are documented out-of-scope extensions (v0.2).

## Cryptographic commitment

This note commits to a single opaque digest of the sealed artifact, signed and anchored in Bitcoin, so anyone can later confirm a disclosed artifact is the one committed here, on this date, without learning anything about it now.

```
SHA-256 digest:   037760b80cce553ea92300a7139b608bdf4f9bd4f0ef0d90cc0f9b5750c11a21
Anchored commit:  d1368b370a6f8a11024fc4fd108a3f4e67a4f1c2
OpenTimestamps:   COMMITMENT.txt.ots  (Bitcoin block 953674, 2026-06-14 19:07:16 UTC,
                  OTS tx d1189cc264634699ce063f9866fff491d3a97c157b7123883a301a3b359965e3)
Signature:        COMMITMENT.txt ssh-signed (namespace: file)
Signer:           softwareengineerasaservant@isurvivable.cv
Signer pubkey:    SHA256:b6D6JxqxZm/brI4tH6N4X+yoEBjeDw4vrtesKmDmhfw  (ed25519)
```

The digest is opaque: it reveals nothing about the proof strategy, the lemma structure, or the bytecode-semantics bridge.  It exists only so that, upon disclosure under engagement, the recipient can confirm the artifact is the one committed and timestamped here.

## Availability

The complete proof bodies, the invariant-decomposition layer, and the bytecode-semantics bridge that ties the abstract proofs to deployed bytecode transfer under a paid scoping engagement, beginning with a billable discovery phase and gated by a one-page mutual NDA.

To engage: softwareengineerasaservant@isurvivable.cv.

## License

The public specification text (these theorem statements and this note) is licensed dual MIT OR Apache-2.0, at your option.  The sealed artifact, proof bodies, and the bytecode-semantics bridge are not covered by this license and are not published.
