# TACITA: Threshold Aggregation without Client Interaction

**Authors:** Varun Madathil, Arthur Lazzaretti, Zeyu Liu, Charalampos Papamanthou (Yale)
**Venue:** ACM CCS 2026 · [IACR ePrint 2025/1579](https://eprint.iacr.org/2025/1579)
**Read:** 2026-11-27 (abstract and author talk only, full PDF not yet read) · Slides: [Paper reading deck, slide 2](https://claude.ai/artifact/WHReNFxEVtywj9CxqT39gt) (private)

## Background
- Secure aggregation (SA) lets a server learn the sum of client inputs but no individual input, even with dropouts.
- Classic SA needs several rounds, per-round setup and client-to-client messages, which is hard for intermittently online devices.
- **OPA** and **Willow** (CRYPTO '25) are one-shot but lack input soundness: the server can manipulate the aggregation, so privacy is DP-level at best.

## Threat model
- A single untrusted server that may tamper with ciphertexts.
- An external **committee** helps with decryption, to prevent residual leakage.
- Clients and committee members may drop out.
- Full privacy assumes a **PKI**. Proven secure in the UC framework.
- ❓ Committee size and corruption threshold: check the full paper.

## Protocol (conceptual, from the abstract; not the exact algorithms)
1. Client i sends **one message**: `ct_i = Enc(x_i)`, `σ_i = Sign(sk_i, x_i)`.
2. The server aggregates homomorphically: `Σ Enc(x_i) = Enc(Σ x_i)`, and `σ = Combine(σ_1, …, σ_n)`. This gives **input soundness**: tampering is detectable.
3. Each committee member sends one share bound to this ciphertext only: `d_j = PartDec(sk_j, ct)`. Then `Σ x_i = Combine({d_j})` once there are ≥ t shares.

**Building blocks:**
- **MKLHTS** (new): succinct multi-key linearly homomorphic threshold signatures.
- An adapted **silent threshold encryption** (Garg et al., CRYPTO '24) with ciphertext-specific decryption and additive homomorphism.

## Properties claimed
| Property | OPA / Willow | TACITA |
|---|---|---|
| One-shot client messages | Yes | Yes |
| Input soundness | No | Yes |
| Constant-size per-client communication | — | Yes |
| Dropout robustness | — | Yes |
| Best privacy | DP-level | Full (with PKI) |

## Limits / relevance
- Centralized and per aggregation: multi-round leakage (Lam 2021, So 2023), sybils (Boenisch 2023) and DFL topologies (Yu 2026) are out of scope.
- Input soundness is about tampered ciphertexts. Whether it also covers model-inconsistency attacks (Pasquini 2022) is still to check in the full paper.
