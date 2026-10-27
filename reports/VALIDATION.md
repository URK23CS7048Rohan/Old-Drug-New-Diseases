# Validation report

All numbers below were measured by the project's own pipeline. Nothing is copied from papers.

## 1. Disease-held-out link prediction (zero-shot)

Test set: 2196 drug–disease pairs (indication + off-label) over 227 held-out diseases. Each true drug is ranked among 2,755 approved drugs, with the disease's other true drugs filtered out. Diseases joined by a direct MONDO link (parent, child or near-duplicate) are always on the same side of the split, so no test disease has such a neighbour in training (checked by `tests/test_data.py`). Brackets: 95% bootstrap CI, resampling diseases.

| Model | MRR | Hits@1 | Hits@10 | Hits@50 | Median rank | AUROC |
|---|---|---|---|---|---|---|
| Random | 0.003 [0.002–0.004] | 0.000 | 0.004 [0.002–0.007] | 0.019 | 1363 | 0.504 |
| Popularity (drug degree) | 0.172 [0.139–0.204] | 0.134 | 0.222 [0.181–0.262] | 0.295 | 279 | 0.853 |
| Network proximity (PPI) | 0.058 [0.037–0.080] | 0.041 | 0.079 [0.052–0.109] | 0.127 | 727 | 0.680 |
| Similar-disease kNN | 0.261 [0.192–0.331] | 0.222 | 0.315 [0.244–0.386] | 0.423 | 97 | 0.877 |
| TransE (PyKEEN, 25 ep.) | 0.040 [0.032–0.048] | 0.014 | 0.089 [0.071–0.107] | 0.226 | 382 | 0.789 |
| RotatE (PyKEEN, 25 ep.) | 0.038 [0.027–0.052] | 0.009 | 0.099 [0.072–0.128] | 0.265 | 378 | 0.780 |
| R-GCN, no pre-trained features, seed 7 | 0.198 [0.141–0.259] | 0.162 | 0.271 [0.209–0.341] | 0.390 | 139 | 0.844 |
| R-GCN, no pre-trained features, seed 13 | 0.213 [0.158–0.272] | 0.178 | 0.287 [0.223–0.354] | 0.407 | 147 | 0.809 |
| R-GCN, no pre-trained features, seed 42 | 0.214 [0.154–0.276] | 0.175 | 0.288 [0.222–0.361] | 0.392 | 144 | 0.855 |
| R-GCN, no pre-trained features, 3-seed ensemble | 0.252 [0.194–0.315] | 0.209 | 0.341 [0.269–0.415] | 0.444 | 106 | 0.860 |
| Hybrid without pre-trained features (ensemble + kNN) | 0.283 [0.214–0.351] | 0.241 | 0.361 [0.286–0.435] | 0.464 | 90 | 0.862 |
| R-GCN + disease/drug features, seed 7 | 0.276 [0.219–0.333] | 0.217 | 0.391 [0.327–0.458] | 0.557 | 32 | 0.908 |
| R-GCN + disease/drug features, seed 13 | 0.271 [0.218–0.328] | 0.225 | 0.355 [0.293–0.416] | 0.557 | 37 | 0.911 |
| R-GCN + disease/drug features, seed 42 | 0.251 [0.199–0.309] | 0.193 | 0.358 [0.297–0.420] | 0.536 | 38 | 0.913 |
| R-GCN + disease/drug features, 3-seed ensemble | 0.284 [0.231–0.342] | 0.226 | 0.390 [0.330–0.451] | 0.582 | 27 | 0.922 |
| **Hybrid Graph Scout: featured R-GCN ensemble + kNN (deployed)** | 0.319 [0.254–0.381] | 0.266 | 0.405 [0.333–0.476] | 0.572 | 29 | 0.921 |

R-GCN + disease/drug features, mean over 3 seeds: MRR 0.266 ± 0.011 (sd), Hits@10 0.368 ± 0.016.

R-GCN, no pre-trained features, mean over 3 seeds: MRR 0.208 ± 0.007 (sd), Hits@10 0.282 ± 0.008.

**A leak we found and removed.** An earlier split held out ontology neighbours only two links around a seed disease. 68 of its 233 test diseases still had a direct MONDO neighbour, often a near-duplicate (*type 1 diabetes mellitus* / *IDDM 1*, *endometrial cancer* / *endometrial carcinoma*), in training. Every number on that split was inflated; the similar-disease kNN baseline, which copies the drugs of the closest training disease, profited most. Earlier-split results are archived in `reports/split_v1/`.

| Model | MRR, leaky split | MRR, corrected split |
|---|---|---|
| Similar-disease kNN | 0.381 | 0.261 |
| Popularity (drug degree) | 0.135 | 0.172 |
| R-GCN + disease/drug features, 3-seed ensemble | 0.349 | 0.284 |
| **Hybrid Graph Scout: featured R-GCN ensemble + kNN (deployed)** | 0.393 | 0.319 |

(The earlier split's R-GCN had no pre-trained features, so its rows compare different models; the kNN and popularity rows compare like with like.)

**Paired comparisons.** Both models scored on the same test pairs; diseases resampled jointly (5,000 bootstrap draws). p is the share of draws in which model A is not better.

| Comparison | MRR difference [95% CI] | p | Hits@10 difference [95% CI] | p |
|---|---|---|---|---|
| deployed hybrid vs similar-disease kNN | +0.058 [+0.036, +0.082] | 0.0000 | +0.091 [+0.059, +0.128] | 0.0000 |
| pre-trained features in the hybrid (with vs without) | +0.037 [+0.018, +0.056] | 0.0000 | +0.044 [+0.021, +0.068] | 0.0002 |
| pre-trained features in the R-GCN ensemble (with vs without) | +0.032 [+0.003, +0.060] | 0.0178 | +0.050 [+0.015, +0.085] | 0.0036 |
| featured R-GCN ensemble alone vs kNN | +0.023 [-0.017, +0.062] | 0.1234 | +0.076 [+0.033, +0.120] | 0.0004 |

## 1b. Which pre-trained features help?

Seed 7, same split and training budget; only the node features differ. Chosen on validation diseases.

| Features | Val MRR | Val Hits@10 | Test MRR | Test Hits@10 | Test median rank |
|---|---|---|---|---|---|
| no pre-trained features | 0.292 | 0.345 | 0.198 [0.141–0.259] | 0.271 | 139 |
| disease text + drug structure (deployed) | 0.294 | 0.364 | 0.276 [0.219–0.333] | 0.391 | 32 |
| disease text + drug structure + protein sequence | 0.283 | 0.353 | 0.241 [0.191–0.295] | 0.342 | 35 |

Disease definitions are embedded after removing every sentence about treatment and masking every drug name, so a held-out disease's text cannot name its drugs. Protein sequence embeddings did not help and are not used in the deployed model.

## 2. Rediscovery of known repurposings

The disease is held out from training entirely. Baricitinib → COVID-19 is **not an edge in PrimeKG**, so it is a prospective test. Ranks are among approved drugs (lower is better).

| Drug → disease | Model | Disease features | Rank |
|---|---|---|---|
| Sildenafil → erectile dysfunction (disease) | held-out ensemble + kNN | PrimeKG graph + disease text, drug structure | **3** / 2745 |
| Thalidomide → plasma cell myeloma | held-out ensemble + kNN | PrimeKG graph + disease text, drug structure | **207** / 2731 |
| Minoxidil → alopecia | held-out ensemble + kNN | PrimeKG graph + disease text, drug structure | **1292** / 2755 |
| Baricitinib → COVID-19 | held-out ensemble + kNN | PrimeKG graph + disease text, drug structure | **20** / 2755 |
| Sildenafil → erectile dysfunction (disease) | production ensemble + kNN | PrimeKG graph + disease text, drug structure | **3** / 2745 |
| Thalidomide → plasma cell myeloma | production ensemble + kNN | PrimeKG graph + disease text, drug structure | **286** / 2731 |
| Minoxidil → alopecia | production ensemble + kNN | PrimeKG graph + disease text, drug structure | **991** / 2755 |
| Baricitinib → COVID-19 | production ensemble + kNN | PrimeKG graph + disease text, drug structure | **21** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s13 | PrimeKG graph + disease text, drug structure | **3** / 2745 |
| Thalidomide → plasma cell myeloma | split_s13 | PrimeKG graph + disease text, drug structure | **344** / 2731 |
| Minoxidil → alopecia | split_s13 | PrimeKG graph + disease text, drug structure | **1599** / 2755 |
| Baricitinib → COVID-19 | split_s13 | PrimeKG graph + disease text, drug structure | **32** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s13 | PrimeKG graph + disease text, drug structure + 27 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **8** / 2745 |
| Thalidomide → plasma cell myeloma | split_s13 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **338** / 2731 |
| Minoxidil → alopecia | split_s13 | PrimeKG graph + disease text, drug structure + 98 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **1543** / 2755 |
| Baricitinib → COVID-19 | split_s13 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **89** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s42 | PrimeKG graph + disease text, drug structure | **5** / 2745 |
| Thalidomide → plasma cell myeloma | split_s42 | PrimeKG graph + disease text, drug structure | **186** / 2731 |
| Minoxidil → alopecia | split_s42 | PrimeKG graph + disease text, drug structure | **995** / 2755 |
| Baricitinib → COVID-19 | split_s42 | PrimeKG graph + disease text, drug structure | **11** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s42 | PrimeKG graph + disease text, drug structure + 27 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **101** / 2745 |
| Thalidomide → plasma cell myeloma | split_s42 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **210** / 2731 |
| Minoxidil → alopecia | split_s42 | PrimeKG graph + disease text, drug structure + 98 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **880** / 2755 |
| Baricitinib → COVID-19 | split_s42 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **24** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s7 | PrimeKG graph + disease text, drug structure | **2** / 2745 |
| Thalidomide → plasma cell myeloma | split_s7 | PrimeKG graph + disease text, drug structure | **253** / 2731 |
| Minoxidil → alopecia | split_s7 | PrimeKG graph + disease text, drug structure | **1438** / 2755 |
| Baricitinib → COVID-19 | split_s7 | PrimeKG graph + disease text, drug structure | **12** / 2755 |
| Sildenafil → erectile dysfunction (disease) | split_s7 | PrimeKG graph + disease text, drug structure + 27 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **7** / 2745 |
| Thalidomide → plasma cell myeloma | split_s7 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **282** / 2731 |
| Minoxidil → alopecia | split_s7 | PrimeKG graph + disease text, drug structure + 98 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **903** / 2755 |
| Baricitinib → COVID-19 | split_s7 | PrimeKG graph + disease text, drug structure + 100 Open Targets genes (genetics/expression/pathways/animal models; no drug or literature evidence) | **23** / 2755 |

## 3. Ablation of evidence agents

Top-20 graph candidates of 80 randomly chosen test diseases are re-ranked; true drugs outside the top-20 keep their graph rank. The Trial Agent is off. Judge weights were fitted on 80 validation diseases: `{"G": 1.6388, "L": 0.3612, "E": 0.0, "D": 0.4}`, λs = 0.0, λc = 0.6323.

| Variant | MRR (seed 7) | MRR (seed 13) | MRR (seed 42) | Hits@1 (mean) | Hits@10 (mean) |
|---|---|---|---|---|---|
| graph only | 0.345 [0.224–0.466] | 0.346 [0.237–0.462] | 0.325 [0.215–0.445] | 0.280 | 0.454 |
| + literature | 0.350 [0.230–0.470] | 0.350 [0.240–0.466] | 0.332 [0.222–0.451] | 0.286 | 0.458 |
| + gene expression | 0.345 [0.224–0.466] | 0.346 [0.237–0.462] | 0.325 [0.215–0.445] | 0.280 | 0.454 |
| + literature + gene | 0.350 [0.230–0.470] | 0.350 [0.240–0.466] | 0.332 [0.222–0.451] | 0.286 | 0.458 |
| full Judge (+ safety, Skeptic conflicts) | 0.349 [0.229–0.469] | 0.350 [0.241–0.466] | 0.332 [0.222–0.451] | 0.284 | 0.458 |

Coverage: 39% of candidate pairs have ≥1 co-mentioning PubMed abstract; 0.1% have L1000 evidence (7 of 80 diseases have a GEO signature). Docking is not part of the ablation (≈20 Vina runs per disease are too costly at this scale); it is run per disease in the app.

**Leakage warning.** Literature co-mention rediscovers what is already published. Known indications are, by construction, heavily published, so the literature channel inflates validation metrics. It is reported separately here, and in the app it is one channel among several.

## 4. Trial cross-check

Top-10 NOVEL graph predictions (PrimeKG indications removed) for held-out test diseases vs 10 random approved drugs; a registered trial (any phase/status) matching the condition and intervention counts as a hit.

| Set | Share with a registered trial | 95% CI | pairs |
|---|---|---|---|
| R-GCN top-10 novel | 26.3% | 19.5%–34.0% | 712 |
| 10 random approved | 4.3% | 2.3%–6.5% | 770 |

## 4b. Direction-of-effect check

Top-20 deployed candidates for each test disease. A candidate can be checked when ChEMBL records an inhibitor/activator action on a target whose direction in the disease is known (expert-curated loss/gain-of-function genetics; OncoKB oncogene/tumour-suppressor roles for cancers). Subtype-dependent cases (genes pointing both ways) are reported as mixed and not counted.

| Candidates | checked | flagged "wrong direction" | "right direction" |
|---|---|---|---|
| true indications | 28 of 882 | 29% | 71% |
| other candidates | 61 of 3798 | 21% | 79% |

Coverage is small, so the check is shown as a warning and does not change scores. With every Open Targets direction label (not only curated ones) it did not separate the groups at all, because variant-consequence labels call oncogenes such as EGFR and JAK2 loss-of-function.

## 4c. Blood-brain-barrier estimate

Physicochemical CNS rules (Hitchcock & Pennington 2006, one rule may be missed) plus LAT1/GLUT1/MCT1 transporter substrates. Against the ATC classification of approved drugs: 95.8% of 337 nervous-system drugs are judged brain-penetrant, against 57.3% of 1552 other drugs. Shown as a warning for nervous-system diseases; it does not change scores.

## 5. Rare diseases and data poverty

Deployed hybrid Graph Scout on the same test pairs, grouped by disease.

| Group | diseases | pairs | MRR | Hits@10 |
|---|---|---|---|---|
| rare (Orphanet) | 87 | 596 | 0.498 [0.372–0.615] | 0.601 [0.478–0.702] |
| common / not in Orphanet | 140 | 1600 | 0.253 [0.195–0.320] | 0.333 [0.261–0.416] |
| ≤ 10 disease genes in graph | 156 | 1104 | 0.213 [0.163–0.275] | 0.325 [0.255–0.412] |
| > 10 disease genes in graph | 71 | 1092 | 0.427 [0.323–0.546] | 0.486 [0.378–0.608] |

## 6. What this does not show

- PrimeKG has no timestamps, so a strict time-split is not possible; the disease-held-out split with ontology-neighbour grouping is used instead.
- Related diseases that are *not* MONDO neighbours (e.g. psoriasis vs psoriatic arthritis) can still sit on opposite sides of the split.
- PrimeKG indications are incomplete: a top-ranked drug that is not a recorded indication is not necessarily wrong.
- Docking with rigid receptors and Vina scores are coarse; decoy z-scores are only a sanity filter.
- The precomputed rare-disease atlas uses the graph scout only; the full agent pipeline runs per disease in the app.
- Nothing here is clinical evidence.

## Figures

![fig_ablation.png](fig_ablation.png)
![fig_hits10.png](fig_hits10.png)
![fig_mrr.png](fig_mrr.png)
![fig_recall_at_k.png](fig_recall_at_k.png)
![fig_training.png](fig_training.png)
![fig_trials.png](fig_trials.png)
