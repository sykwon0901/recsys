# Adaptive Personalization in Retrieval-based Recommendation

This repository accompanies the master's thesis *Adaptive Personalization in Retrieval-based Recommendation*, completed in the Master in Applied Data Science programme at Frankfurt School of Finance & Management.

The study examines where user history should enter a non-generative e-commerce retrieve-then-rerank pipeline: during candidate retrieval, during reranking, or at both stages. It uses a leakage-controlled held-out-item benchmark built from two Amazon Reviews 2023 categories:

- **Facial Skincare (Skincare):** primary empirical category; 2,288 balanced benchmark cases.
- **Herbal Supplements (Supplements):** secondary cross-category setting; 1,968 balanced benchmark cases.

The repository contains the executed experiment and analysis notebooks. Their stored outputs provide an auditable record of the reported pipeline. The Amazon source data and large local artifacts are not redistributed.

## Research design

Each benchmark case contains one target review per user, a synthetic query derived under target-cue controls, and only the user's interactions that occurred strictly before the target. The target parent item is excluded from the user's effective history. Candidate retrieval and reranking are evaluated under frozen within-category interfaces and user-group-disjoint out-of-fold estimation.

The study addresses five questions:

1. Does history-conditioned retrieval improve candidate exposure over query-only retrieval?
2. Does supervised reranking improve top-rank quality on a fixed query-only candidate pool?
3. What incremental value does personalization provide at retrieval and reranking when each is activated separately?
4. Do the two stages each add value when the other is active, and is their joint effect complementary?
5. How do these effects vary across category structures and observed history regimes?

Four reranker families are evaluated: a deterministic **Shannon Entropy** heuristic, **LightGBM**, a spline-based **generalized additive model (GAM)**, and a listwise structured-feature **Transformer**.

## Canonical condition terminology

The thesis terminology was simplified after much of the executable pipeline had been built. Notebook filenames, machine keys, paths, manifests, and stored outputs therefore retain legacy identifiers. They are part of the executed lineage and must not be renamed mechanically.

The final thesis-facing names are:

| Final display name | Candidate source | Same-user prior in reranking | Machine series | Common legacy names or filename tokens |
|---|---|---:|---|---|
| **S1-Q** | Query-only retrieval winner; upstream order preserved | Not applicable | Upstream raw condition | `P0`, `s1q_raw` |
| **S1-P** | QCHS-personalized retrieval winner; upstream order preserved | Not applicable | Upstream raw condition | `P1-only`, `s1p_raw` |
| **Base** | Frozen S1-Q pool | No | `a` | `P2-Q`, `S2-Q`, `s2q`, `QQ`, `no_prior_rerank` |
| **RankP** | Same frozen S1-Q pool as Base | Yes, strict pre-target prior | `b01` | `P2-P`, `S2-P`, `s2p`, `QP`, `prior_rerank`; some filenames use `base_rerank` |
| **RetrP** | Frozen S1-P pool | No | `b02` | `S2-Q/P`, `PQ`, `Y10`, `no_prior_on_personalized`, `personalized_noprior_rerank` |
| **Full** | Frozen S1-P pool | Yes, strict pre-target prior | `c` | `personalized_rerank`, `full_rerank`; obsolete documentation may say `transport` or `retrained_full` |

The four reranked conditions form a matched 2 x 2 policy design:

| | No-prior reranker | Prior-aware reranker |
|---|---:|---:|
| Query-only candidate pool | Base | RankP |
| Personalized candidate pool | RetrP | Full |

Base and RankP are fitted on the query-only candidate interface. RetrP and Full are fitted separately on the personalized candidate interface under the corresponding matched contract. No fitted model is reused across candidate sources.

### Additional terminology mapping

| Legacy or ambiguous term | Canonical term and interpretation |
|---|---|
| `QCHA` | **QCHS**: query-conditioned history selection. `QCHA` remains only in immutable legacy code, paths, or machine fields. |
| `Shannon` as a method name | **Shannon Entropy**. The shorter form remains valid only in immutable machine keys such as `shannon_rerank` or in the citation to Shannon (1948). |
| fixed-model transport; transfer contract | **Matched own-pool fitting** or **own-pool estimation**. |
| point-in-time preference evidence | **Point-in-time reviewed-item evidence**. |
| `GAM-lite` in notebook headings | **GAM** in the thesis-facing method name. |
| attribute | A source-level product field or a generally described product characteristic. |
| facet family / facet value | A normalized semantic role / a normalized concept within that role. |
| feature | A numeric or categorical model input derived from the registered evidence. |

**Brand policy.** Brand is excluded only from synthetic-query construction because the target brand can act as a direct lookup cue. Brand remains available in item representations, graph construction, user profiles, personalized retrieval, reranking features, and candidate representations. It is treated as a separate preference facet, not as a product-functional facet.

## Evaluation protocol

- **Stage 1 headline endpoint:** Recall@1,000. Under the single-positive protocol, this is numerically identical to HitRate@1,000, but the term *Recall* is used to distinguish candidate exposure from final ranking accuracy.
- **Stage 2 headline endpoint:** unconditional NDCG@5 at candidate depth 1,000.
- **Sensitivity depths:** deterministic prefixes of 100, 300, 500, and 700 from the same exported 1,000-item candidate interfaces.
- **Missing target:** a target absent from the evaluated candidate prefix receives zero; it is not removed from the denominator.
- **Primary inferential contrast:** RankP - Base within each reranker family.
- **Complementarity criterion:** Full must improve over both RankP and RetrP. The 2 x 2 interaction is reported descriptively and does not by itself establish complementarity.

The final thesis applies Holm correction to the four Tier-1 RankP - Base tests within each category. A fixed, gated Tier-2 sequence is entered only when the primary LightGBM Tier-1 contrast is rejected: Full - RankP is tested first, followed by Full - RetrP only if the first Tier-2 test is rejected.

## Repository structure

```text
recsys/
├── README.md
├── .gitignore
└── categories/
    ├── facial_skincare/
    │   └── [executed notebooks]
    └── herbal_supplements/
        └── [executed notebooks]
```

The repository is intentionally notebook-centred. Empty software-package folders are not required. Compact, publication-safe manifests or final tables may be added only when they directly support a thesis result and do not duplicate large notebook outputs.

### Notebook guide

Notebook numbering is category-specific after the shared experimental core. Follow the numeric order inside each category and treat the notebook's actual filename as its lineage identifier.

| Notebook range | Role |
|---|---|
| `01-02` | Source-data checks, category schema, and item/facet representation |
| `03-05` | Temporally valid target sampling, retrieval artifacts, and historical-review signal construction |
| `06` | Leakage-controlled synthetic-query construction |
| `06x` | Query-formulation comparison where present |
| `07-09` | Query-only retrieval selection, QCHS-personalized retrieval, and frozen candidate-pool export |
| `10a-10c` | Shannon Entropy reranking conditions |
| `11a-11c` | LightGBM reranking conditions |
| `12a-12c` | GAM reranking conditions |
| `13a-13c` | Transformer reranking conditions |
| `14` | Canonical per-case aggregation and rank-derived metric layer |
| `15` | Descriptive stage-allocation summary |
| `16` | Stored paired-inference analysis; see the authority note below |
| `17` | Stage bottleneck and interaction diagnostics |
| `18-19` | LightGBM and Transformer interpretation |
| `20+` | Category-specific controls, alignment, mechanism, regime, cross-category, and efficiency diagnostics |

### Reranker filename mapping

For LightGBM, GAM, and Transformer, the executable branch is identified by both its notebook series and its candidate-pool path:

| Series | Typical filename token | Thesis condition |
|---|---|---|
| `a` | `no_prior_rerank` | Base |
| `b01` | `prior_rerank` or legacy `base_rerank` | RankP |
| `b02` | `personalized_noprior_rerank` | RetrP |
| `c` | `personalized_rerank` or `full_rerank` | Full |

Shannon Entropy is deterministic and has no separately fitted RetrP notebook. Its Base condition preserves S1-Q order, and its RetrP condition preserves S1-P order.

## Analytical authority

The evidence hierarchy is:

1. Frozen Stage 1 candidate exports define the S1-Q and S1-P interfaces.
2. Notebook 14 is the canonical authority for per-case target ranks and rank-derived metrics.
3. Notebook 15 supplies descriptive stage-allocation summaries.
4. The final thesis defines the inferential hierarchy and the claims attached to each comparison.
5. Later notebooks provide mechanism, robustness, interpretation, and efficiency diagnostics; they do not replace the headline endpoints.

Some stored Notebook 16 executions retain an earlier merged eight-test Holm family, combining RankP - Base and Full - RankP across four rerankers. Those adjusted p-values and rejection labels are an archived analysis state, not the authority for the final thesis protocol. The final four-test Tier-1 family and gated LightGBM Tier-2 sequence described above govern the reported confirmatory conclusions.

## Data and artifact availability

The source reviews and item metadata come from the public [Amazon Reviews 2023](https://amazon-reviews-2023.github.io/) dataset released by the McAuley Lab.

The following are not redistributed:

- Amazon review and item-metadata source files;
- processed item, review, facet, profile, and benchmark tables;
- dense or sparse indices, embeddings, graphs, and query caches;
- frozen candidate-pool parquet files;
- per-candidate predictions and bulk CSV exports;
- fitted model files and checkpoints;
- local execution logs.

In this repository, *artifact* is a broad term. It may refer to a processed dataset, candidate pool, query cache, model file, prediction export, metric table, manifest, or figure input. It does not mean that every such file must be published.

Where a manifest records a **SHA-256** value, the value is a 64-character cryptographic fingerprint of the exact file bytes. It supports identity checks: the same file produces the same fingerprint, while any byte-level change produces a different value. A hash neither contains nor reconstructs the artifact. Large excluded artifacts therefore do not need to be uploaded merely because a notebook or manifest records their hashes.

## Reproducibility scope

The executed notebooks and their stored outputs are provided as an audit-ready evidence package. This is not a one-command, raw-data-inclusive reproduction bundle. A complete rerun requires:

1. obtaining the original Amazon Reviews 2023 category data;
2. restoring the local directory structure expected by the notebooks;
3. generating the omitted intermediate artifacts in numeric notebook order;
4. preserving the frozen case IDs, target timestamps, candidate interfaces, fold assignments, feature registries, seeds, and machine keys;
5. executing the category-specific reranking branches before canonical aggregation.

The constrained DSPy/DeepSeek step in Notebook 06 performs linguistic normalization under fixed acceptance and fallback rules. The archived accepted queries and their quality-control fields, rather than a fresh API response, define the executed benchmark.

The notebooks contain the method-specific setup cells and fixed experimental settings used for the reported runs. Compute requirements vary by branch: most processing and tabular reranking were executed on CPU resources, while the Transformer branch used GPU resources. Runtime records are descriptive and do not support a hardware- and scope-matched deployment-cost ranking across methods.

## Main findings

- Personalized retrieval increased candidate exposure in both categories, but it did not produce a statistically supported improvement in headline top-five quality.
- The clearest evidence for useful same-user history appeared in prior-aware reranking, although the inferential pattern differed by category and reranker family.
- Refitting the prior-aware reranker on the personalized pool did not yield a statistically supported incremental effect under the prespecified criterion.
- The complementarity criterion was not met. This is a non-detection, not proof of redundancy, equivalence, or non-complementarity.
- Greater observed history depth did not reliably imply larger personalization gains.

The conclusions are specific to the implemented stage policies, candidate interfaces, two selected categories, synthetic-query benchmark, and offline held-out-item task. They should not be interpreted as an online production evaluation or as evidence that retrieval-side personalization is generally ineffective.

## Thesis reference

Kwon, Seon Young (2026). *Adaptive Personalization in Retrieval-based Recommendation*. Master's thesis, Frankfurt School of Finance & Management.
