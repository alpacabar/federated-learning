# TACITA: Threshold Aggregation without Client Interaction

**Authors:** Varun Madathil, Arthur Lazzaretti, Zeyu Liu, Charalampos Papamanthou (Yale)
**Venue:** ACM CCS 2026, pp. 2461–2474 · [doi:10.1145/3830454.3832744](https://doi.org/10.1145/3830454.3832744) · [ePrint 2025/1579](https://eprint.iacr.org/2025/1579) (full version) · CC BY 4.0
**Read:** 2026-11-27 (full CCS paper) · Slides: [Paper reading deck, slide 2](https://claude.ai/artifact/WHReNFxEVtywj9CxqT39gt) (private)

## Problem
One-shot secure aggregation: each client sends **one** message and goes offline, with a single server. Four challenges:
1. **Residual leakage** (Halevi et al. 2011). Without interaction, the server can aggregate arbitrary subsets: `S1 = x1+x2+x3`, `S2 = x1'+x2+x3` ⇒ `x1 = S1 − S2 + x1'`. The fix is an external **committee** that decrypts only once per round.
2. **Scalability.** OPA's committee costs O(log N). Willow's committee is O(1) but runs a **DKG every round**.
3. **Input soundness.** The committee must know that enough *genuine* client ciphertexts were aggregated. **Willow has none:** the server can discard all but the target's ciphertexts, fabricate the rest and isolate the target (disclosed to the authors; OPA was updated). OPA has it via per-client signatures, at O(N) or O(log N) communication.
4. **Dropouts** of clients and committee members.

## Threat model & assumptions
- **Malicious server** that can collude with up to a threshold of clients and committee members. Privacy holds if the server corrupts only a **minority** of clients and committee. Static corruption, UC security, programmable global random oracle.
- **PKI** (without it, man-in-the-middle attacks on input soundness are possible).
- **Trusted CRS**, for pairings and KZG-style proofs. OPA and Willow don't need this.
- An external, public client-selection algorithm (as in Flamingo).

## Protocol (Fig. 1 / Fig. 3)
1. **Client:** mask `x̂_i = x_i + H(rand_i)`, encrypt `x̂_i` with the modified STE and `rand_i` with the original STE, and sign the 6th ciphertext component with MKLHTS. One message to the server.
2. **Server:** add the ciphertexts homomorphically, and aggregate the signatures into `σ*` with a proof of ≥ t signers.
3. **Committee:** check `MTS.Verify(..., σ*, t, vk) = 1`. If it passes, return ciphertext-bound partial decryptions.
4. **Server:** `DecAggr`, then recover each `rand_i`, subtract `Σ s_i`, and take a small **discrete log** to get `Σ x_i`.

### Key math
- Base encryption (witness encryption for BLS / Boneh–Franklin IBE): `ct1 = r·[1]_1`, `ct2 = r·(pk ∘ H(tag)) + m`. It is additively homomorphic and supports key aggregation.
- **Ciphertext binding:** use `part-dec_j = sk_j · ct1` instead of `sk_j · H(tag)`. Then `Σ x_i = ct2 − Σ_j part-dec_j ∘ H(tag)`, so shares can't decrypt an individual ciphertext.
- **MKLHTS signature:** `σ_i = sk_i · (H(tag) + [s_i])`, verified by `(vk_i ∘ H(tag)) + (vk_i ∘ S_i) = [1] ∘ σ_i`.
  - Aggregate: `σ* = Σ σ_i`, `S* = Σ [s_i]`, `vk* = Σ vk_i`.
  - The naive check fails because `Σ sk_i s_i ≠ (Σ sk_i)(Σ s_i)`. Fixed with a generalized sumcheck (hinTs): `SK(X)·B(X) = vk* + Q_Z(X)Z(X) + Q_x(X)X`, where `B(X)` encodes the signer set (bits, sum ≥ t).
- MKLHTS is unforgeable and extractable in the AGM under discrete log (Thm 1). The protocol UC-realizes `F_agg` (Thm 2).

## Comparison
| Scheme | PKI | CRS | Input soundness | Server→committee | Per-round setup | One-shot |
|---|---|---|---|---|---|---|
| Willow | No* | No | No (vulnerable) | O(N) | DKG | ✓ |
| OPA | Yes | No | Yes (signatures) | O(log N) | one-time | ✓ |
| **TACITA** | Yes | Yes | Yes (MKLHTS) | **O(1)** | one-time | ✓ |

## Performance (Rust, Apple M2, BLS12-381, research-grade)
| N | Client total | Server agg | Server dec | Committee verify / part-dec |
|---|---|---|---|---|
| 32 | 63.6 ms | 21.9 ms | 14.3 ms | 8.8 ms / 0.89 ms |
| 1024 | 32.6 s | 0.42 s | 0.18 s | 8.8 ms / 0.89 ms |

- Messages are constant in N: client→server 1.74 kB, server→committee 2.15 kB, committee→server 0.18 kB.
- At N = 512, client→server is 1.2 kB vs 400.1 kB for OPA.
- Client cost is dominated by MKLHTS hint generation, which can be precomputed if the cohort is known (expected ~10× reduction).

## FL-specific notes
- **Model inconsistency** (Pasquini 2022): mitigated by setting `tag = H(iteration | model)`. If clients use different tags, decryption and signature verification fail.
- **Poisoning / input validation:** not handled. Could be combined with FedOpt or bRSA, or ZK proofs of well-formedness; left as future work.
- **Small input domain** because of the final discrete log, extended by chunking (B ≤ 4 for 32-bit floats). Vector inputs are handled with KAHE, described in the full version.

## Relevance to my work
- Covers the server-tampering and model-inconsistency attacks, but it's still **per aggregation** and centralized. Multi-round participation leakage (Lam 2021, So 2023), sybils beyond the minority threshold (Boenisch 2023) and DFL topology attacks (Yu 2026) are out of scope.
- Hierarchical FL: could an edge aggregator act as the "server" with a per-cluster committee? The CRS, PKI and hint costs per cluster would need checking.
