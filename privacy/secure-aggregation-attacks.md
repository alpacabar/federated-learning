# Breaking Secure Aggregation in CFL and DFL

> **Claim:** Secure Aggregation (SA) on its own doesn't guarantee privacy in either Centralized FL (CFL) or Decentralized FL (DFL). Attackers get around it through weaknesses in the **structure** (topology), at the **node** level (malicious server or clients), and at the **aggregation** level (participation patterns and time).

_Last updated: 2026-11-27_

---

## 1. Taxonomy

### A. Network & structural exploitation (DFL)

| Attack | Mechanism | Adversary | Rounds | Key paper |
|---|---|---|---|---|
| **Topological asymmetry** | Sparse graphs give colluding nodes uneven, linearly independent views of their neighborhoods. Recovering the hidden mixing weights and honest states is cast as **mHSSP** (binary/uniform weights) or **mHLCP** (bounded non-uniform weights) and solved with lattice reduction plus structural filtering. | Colluding semi-honest nodes | 1 (exact only at t=0) | Yu, Ji, Bjerva, Li — *When Topology Betrays Privacy* (CCS 2026; arXiv 2609.08476) |
| **Neighbor isolation ("trivial attack")** | A corrupted node with exactly **one** honest neighbor subtracts its own and the other corrupted contributions, which leaves that neighbor's state. In directed (Push-Sum) graphs the unknown weight narrows this to a finite candidate set. | ≥1 corrupted node | 1 | same as above (pre-pruning step) |
| **Aggregate differencing / generalization leakage** | Subtract two aggregates, one that includes an honest node and one that excludes it. A neighbor can also exploit local generalization leakage. | Passive or active neighbor | 1+ | Pasquini, Raynal, Troncoso — *On the (In)security of P2P Decentralized ML* (IEEE S&P 2023) |

**Conditions noted in Yu et al.:**
- Necessary: |corrupted| ≥ |honest nodes in core subgraph|.
- Needs topological asymmetry: a fully connected graph with uniform weights is rank-deficient and has no unique solution.
- Needs topology priors (who the neighbors are, degrees) to filter out spurious candidates.
- Permutation ambiguity remains, so a recovered update can be matched to the wrong honest node.

### B. Node / server manipulation (CFL)

| Attack | Mechanism | Key paper |
|---|---|---|
| **Model inconsistency** | A malicious server sends non-identical models to different clients, so the aggregate isolates the target in a single round. Works even against *ideal* SA. | Pasquini, Francati, Ateniese — *Eluding SA via Model Inconsistency* (CCS 2022) |
| **Sybil / malicious device injection** | The server injects devices it controls, which strips the target's masking "crowd". Also defeats distributed DP. | Boenisch et al. — *Reconstructing Individual Data Points in FL Hardened with DP and SA* (arXiv 2301.04017, 2023) |
| **Trap weights / imprint modules** (related) | Changes to model weights or architecture make the gradients invertible, and the attack scales under SA. | Boenisch et al. *When the Curious Abandon Honesty*; Fowl et al. *Robbing the Fed* (ICLR 2022); Zhao et al. **LOKI** (S&P 2024); Scale-MIA (NDSS 2025); SEER (Garov et al., arXiv 2306.03013) |

### C. Aggregation, participation & temporal exploitation (CFL & DFL)

| Attack | Setting | Mechanism | Key paper |
|---|---|---|---|
| **Gradient disaggregation** | CFL | Uses partial participation and device-analytics side channels to rebuild the **binary participant matrix**, then factorizes it to disaggregate individual updates. | Lam et al. (ICML 2021) |
| **Multi-round SA leakage** | CFL | Random user selection leaks individual models within a number of rounds linear in the number of users. Proposed fix: structured selection. | So et al. — *Multi-RoundSecAgg* (AAAI 2023) |
| **Stationary inactive nodes** | Async DFL | Inactive nodes keep the same state across rounds, giving colluders fixed reference points. They solve a linear system over many summations. Success depends on graph **girth**. | Dekker, Erkin, Conti (PoPETs 2025) |
| **Quasi-static gradients (no SA)** | DFL (gossip / D-GD) | Each received message is treated as an equation. Reconstructs data from **non-neighbors** across rounds. | El Mrini, Cyffers, Bellet (ICML 2024) |
| **Activation sparsity + data priors** | CFL | Uses ReLU activations in the first FC layer of aggregated models to recover samples **and group them by client** across rounds. | Marchand et al. — **SRATTA** (ICML 2023) |
| **Blind source separation** (related) | CFL | Aggregate FC gradients are linear mixtures of inputs, so ICA can unmix them (batches up to 1024). | Kariyappa et al. — *Cocktail Party Attack* (ICML 2023) |
| **Membership inference under SA** (related) | CFL | Treats SA as an LDP mechanism with correlated noise and shows weak privacy even in a single round. | Ngo et al. (ECML-PKDD 2024) |

---

## 2. Defenses mapped to attacks

| Defense | Counters | Status / caveat |
|---|---|---|
| Model consistency checks (signatures / MACs on the broadcast model) | Model inconsistency (B) | Proposed by Pasquini et al. and in schemes such as RLSA-PFL (arXiv 2502.08989). Doesn't stop sybils. |
| Sybil resistance / verified participant sets | Device injection (B) | Largely open. Boenisch et al. argue the root cause is the server's power imbalance. |
| Structured user selection | Participation / multi-round (C) | So et al. (AAAI 2023). Trades privacy against participation. |
| High-girth / acyclic topologies | Async DFL reconstruction (C) | Dekker et al.: exact attacks over summations are impossible on acyclic graphs. Not evaluated against lattice attacks. |
| Central (aggregate-level) DP | Lattice attack (A) | Yu et al.: breaks exact recovery. A noise-tolerant variant still recovers about 25–50% of the true vectors. |
| Local DP | Lattice attack (A) | Weak: the algebraic structure survives, so protection depends only on noise magnitude. |
| Info-theoretic decentralized SA | All of (A), in theory | Zhang et al. (arXiv 2508.00596). Rate bounds only, with no evaluation against real attacks. |
| One-shot SA with input soundness (TACITA) | Server tampering with ciphertexts (CFL) | Madathil et al. (CCS 2026). Single server plus a decryption committee; assumes a PKI. Per-aggregation only, so multi-round leakage, sybils and DFL topologies are out of scope. |
| Non-forward secure aggregation (NFSA) | — | Zhou (CCS 2026). Single-server SA via two-layer secret sharing; not yet read. |

---

## 3. Gap analysis

Legend: 🔴 saturated / overstudied · 🟡 moderate · 🟢 understudied · ⚪ nothing found yet

| Area | Level | Evidence |
|---|---|---|
| Malicious-server model manipulation in CFL (trap weights, imprints, LOKI, SEER, Scale-MIA) | 🔴 | Many attacks since 2021, plus a USENIX Sec '25 SoK. The new contribution now is mostly **detectability** and defenses. |
| Gradient inversion of aggregated CFL updates (passive) | 🔴 | CPA, SRATTA, HSSP-based perfect inversion (Li et al. 2024). |
| Multi-round / partial-participation leakage in CFL | 🟡 | Lam 2021 and So 2023 are foundational. A rigorous composition with DP accounting is thin. |
| Passive reconstruction in DFL **without** SA | 🟡 | Pasquini 2023, Mrini 2024, Ji 2025 (info-theoretic). Yu et al. 2024 argue the opposite (DFL is provably *more* private). The debate is unresolved. |
| DFL **with** SA: topology-driven attacks | 🟢 | Essentially just Yu et al. (CCS 2026; single-round, exact) and Dekker 2025 (async, summation). |
| Multi-round lattice attacks in DFL+SA (beyond t=0) | 🟢 | Listed as an explicit limitation by Yu et al. |
| Directed / Push-Sum and **time-varying** topologies | 🟢 | Directed graphs are covered only formally in Yu et al. For dynamic topologies there's only an EPFL student report suggesting they *help* privacy. |
| Noise-tolerant lattice attacks vs central DP | 🟢 | Future work in Yu et al. The DP × lattice interaction is unexplored. |
| **Active/malicious** nodes in DFL+SA (a DFL analog of model inconsistency: sending different states to different neighbors) | ⚪ | Pasquini 2023 covers active DFL adversaries but not under SA. I found nothing combining the two. |
| **Hierarchical FL (client–edge–cloud) + SA** privacy attacks | ⚪ | HFL+SA work is protocol and info-theoretic rate design (e.g. H-SecCoGC, HSA rate regions). I found no attacks that exploit an edge aggregator or the cluster structure. ← relevant to ITRC 2세부 (HFL) |
| Combining temporal priors (Mrini) with SA in DFL | ⚪ | Mrini assumes no SA and Yu is single-round. Nobody has merged them. |
| Defenses that are both girth-aware **and** asymmetry-aware | ⚪ | Dekker's girth defense hasn't been tested against the lattice attack. |
| Sybil-resistant participant verification for SA | 🟢 | Flagged as open by Boenisch et al. |

**Caveat:** ⚪ means "not found in a targeted search on 2026-11-24", not "proven absent". Check Google Scholar citation trees for Yu et al. 2026 and Dekker 2025 before claiming novelty.

---

## 4. Candidate research directions

1. **Lattice attacks on hierarchical FL + SA.** An edge aggregator, or colluding clients inside a cluster, see asymmetric partial sums much like sparse DFL. Check whether cluster assignment works like a topology. *(Ties directly to the ITRC HFL work.)*
2. **Multi-round mHSSP/mHLCP.** Extend Yu et al. past t=0 using Mrini-style temporal equations, in effect stacking the observation matrices across rounds.
3. **Active DFL+SA adversary.** A node that sends inconsistent states to different neighbors to manufacture the asymmetry the lattice attack needs, even in dense graphs.
4. **Topology + DP co-design defense.** Pick graph girth and mixing weights to guarantee rank deficiency against k colluders, then add minimal central DP. This could connect to the Fed RDT rate-distortion framing (privacy cost vs distortion).

---

## 5. Reading list

| # | Paper | Venue | Link |
|---|---|---|---|
| 1 | Yu, Ji, Bjerva, Li — When Topology Betrays Privacy | CCS 2026 | https://arxiv.org/abs/2609.08476 |
| 2 | Pasquini, Francati, Ateniese — Eluding SA via Model Inconsistency | CCS 2022 | https://arxiv.org/abs/2111.07380 |
| 3 | Pasquini, Raynal, Troncoso — On the (In)security of P2P Decentralized ML | S&P 2023 | https://arxiv.org/abs/2205.08443 |
| 4 | Boenisch et al. — Reconstructing Individual Data Points in FL Hardened with DP and SA | arXiv 2023 | https://arxiv.org/abs/2301.04017 |
| 5 | Boenisch et al. — When the Curious Abandon Honesty | — | https://arxiv.org/abs/2112.02918 |
| 6 | Lam et al. — Gradient Disaggregation | ICML 2021 | https://arxiv.org/abs/2106.06089 |
| 7 | Dekker, Erkin, Conti — Topology-Based Reconstruction Prevention for Decentralised Learning | PoPETs 2025 | https://arxiv.org/abs/2312.05248 |
| 8 | El Mrini, Cyffers, Bellet — Privacy Attacks in Decentralized Learning | ICML 2024 | https://arxiv.org/abs/2402.10001 |
| 9 | Marchand et al. — SRATTA | ICML 2023 | https://arxiv.org/abs/2306.07644 |
| 10 | So et al. — Securing Secure Aggregation (Multi-RoundSecAgg) | AAAI 2023 | https://arxiv.org/abs/2106.03328 |
| 11 | Kariyappa et al. — Cocktail Party Attack | ICML 2023 | https://arxiv.org/abs/2209.05578 |
| 12 | Zhao et al. — LOKI | S&P 2024 | https://arxiv.org/abs/2303.12233 |
| 13 | Fowl et al. — Robbing the Fed | ICLR 2022 | https://arxiv.org/abs/2110.13057 |
| 14 | Ngo et al. — SA Is Not Private Against Membership Inference | ECML-PKDD 2024 | https://arxiv.org/abs/2403.17775 |
| 15 | Ji et al. — Re-Evaluating Privacy in Centralized and Decentralized Learning | ICASSP 2025 | https://arxiv.org/abs/2409.14261 |
| 16 | Yu, Li et al. — Provable Privacy Advantages of DFL | arXiv 2024 | https://arxiv.org/abs/2407.09324 |
| 17 | Li et al. — Perfect Gradient Inversion via HSSP | arXiv 2024 | https://arxiv.org/abs/2409.14260 |
| 18 | Zhang et al. — Info-Theoretic Decentralized SA with Collusion Resilience | arXiv 2025 | https://arxiv.org/abs/2508.00596 |
| 19 | Sultan et al. — RLSA-PFL (model-inconsistency detection) | arXiv 2025 | https://arxiv.org/abs/2502.08989 |
| 20 | Carletti et al. — SoK: Gradient Inversion Attacks in FL | USENIX Sec 2025 | https://www.usenix.org/system/files/usenixsecurity25-carletti.pdf |
| 21 | Madathil, Lazzaretti, Liu, Papamanthou — TACITA: Threshold Aggregation without Client Interaction | CCS 2026 | https://eprint.iacr.org/2025/1579 |
| 22 | Zhou — NFSA: Non-Forward Secure Aggregation with One Server via Two Layer Secret Sharing | CCS 2026 | — |

### Notes on attributions in the original summary
- The "aggregate differencing / local generalization leakage" point is from **Pasquini et al. S&P 2023** (the DFL paper), not the CCS 2022 model-inconsistency paper.
- "Malicious device injection" is from Boenisch et al.'s **sybil-device** paper (arXiv 2301.04017). Their earlier *Curious Abandon Honesty* paper uses trap weights instead.
- The mHSSP/mHLCP and trivial-attack details come from Yu et al. 2026.
