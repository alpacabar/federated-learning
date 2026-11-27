# When Topology Betrays Privacy: Lattice-Based Reconstruction Attacks on Secure Aggregation in DFL

**Authors:** Wenrui Yu, Changlong Ji, Johannes Bjerva, Qiongxiu Li (Aalborg U. / IP Paris)
**Venue:** ACM CCS 2026 · [arXiv 2609.08476](https://arxiv.org/abs/2609.08476)
**Read:** 2026-11-27 · Slides: [Paper reading deck, slide 1](https://claude.ai/artifact/WHReNFxEVtywj9CxqT39gt) (private)

## Background
- In decentralized FL, secure aggregation (SA) is **local**: each node only learns the weighted sum of its neighbours' states.
- In **sparse graphs**, neighbourhoods overlap unevenly, so colluders see different linear mixes of the same honest states.
- Both the states and the mixing weights are hidden, which makes recovery a **Hidden Subset Sum Problem**.

## Threat model
- Colluding **semi-honest** nodes pool their views.
- They know topology priors (neighbour IDs, degrees), but not honest states or weights.
- Single round; exact recovery only at t = 0 (known initialization).
- Necessary condition: colluders ≥ honest nodes in the core subgraph, and the topology must be asymmetric (a fully connected graph with uniform weights is rank-deficient).

## Attack, step by step
1. **Observe:** colluder i sees `s_i = Σ_{j ∈ N(i)} w_ij x_j`.
2. **Stack:** after scaling weights to integers, `H ≡ A · X (mod Q)`.
   - `A ∈ {0,1}` → **mHSSP** (uniform weights)
   - `A ∈ {0,…,c}` → **mHLCP** (bounded non-uniform weights)
3. **Recover:**
   - *Trivial attack:* colluder k with exactly one honest neighbour h computes `x_h = (s_k − Σ_{j ∈ C} w_kj x_j) / w_kh`. In directed (Push-Sum) graphs this narrows to a finite candidate set.
   - Prune those nodes to get the **core subgraph**, then apply **lattice reduction plus structural filtering** to `H` to recover `A`, then `X`.
   - The recovered updates then feed standard gradient inversion.

## Defenses (as reported)
| Defense | Effect |
|---|---|
| Local DP, `A(X+E)` | Weak: the algebraic structure survives |
| Aggregate-level DP, `AX + E` | Breaks exact recovery; a noise-tolerant variant still recovers 25–50% of true vectors |
| Topology design | Avoid single-honest-neighbour nodes; high girth (Dekker et al., PoPETs 2025) |
| Structured participation / info-theoretic DSA | So et al. (AAAI 2023); Zhang et al. (2025) |

## Limitations / open
- Exact recovery only in the first round. Multi-round attacks are future work.
- Permutation ambiguity: a recovered update can be matched to the wrong node.
- Noise-tolerant lattice attacks, and how DP interacts with lattice methods.

## My notes
- Possible extension to **hierarchical FL**: an edge aggregator sees asymmetric partial sums, much like a sparse DFL graph (ITRC 2세부).
- See the gap analysis in [../privacy/secure-aggregation-attacks.md](../privacy/secure-aggregation-attacks.md).
