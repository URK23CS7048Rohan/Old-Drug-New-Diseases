# Model card: Old Drug, New Disease — Graph Scout v2

Structured after Mitchell et al., *Model Cards for Model Reporting* (FAT* 2019). All numbers are measured by this repository's
code on held-out diseases; see [`reports/VALIDATION.md`](reports/VALIDATION.md) for every table and how to reproduce it.

## Model details

| | |
|---|---|
| Developer | Rohan |
| Version | 2.0 (October 2026): pre-trained node features, leak-free disease split |
| Task | Rank all 2,755 approved drugs for a disease by how likely they are to treat it (zero-shot drug repurposing) |
| Architecture | Heterogeneous relational GCN (2 layers, 128 d, 8 bases, 50 edge types) with a DistMult decoder, 3-seed ensemble, fused with a similar-disease kNN (weight 0.1, chosen on validation diseases) |
| Input features | Disease: PubMedBERT embedding of name + definition (treatment sentences removed, drug names masked). Drug: 2048-bit Morgan fingerprint. Graph structure for everything else. |
| Training | Episodic (TxGNN-style), listwise softmax loss over drugs, contraindication auxiliary loss; CPU only, ~20 min per model |
| Evidence layer | Literature (PubMed), expression reversal (LINCS L1000), docking (AutoDock Vina vs decoys), safety (openFDA, SIDER), trials; Skeptic with direction-of-effect and blood-brain-barrier warnings; Judge with weights fitted on validation diseases |

## Intended use

- **Hypothesis generation for research**: shortlisting approved drugs worth testing in the laboratory for a disease, especially rare and neglected diseases with no treatment.
- **Teaching and exploration** of biomedical knowledge graphs.
- Users: researchers, students, educators. Every candidate is shown with its evidence and the Skeptic's objections so a person can judge it.

## Out of scope

- **Any clinical decision**: choosing a treatment, dose or patient. Outputs are untested hypotheses, not medical advice.
- Ranking drugs that are not approved (the candidate list is approved drugs only), predicting doses, toxicity or efficacy sizes.
- Regulatory or reimbursement use.

## Training data

PrimeKG (Chandak et al., *Sci. Data* 2023): 129,375 nodes, 8.1 M edges, 17 sources. Labels are drug–disease *indication* and
*off-label use* edges (DrugBank-derived). Disease text from MONDO/Orphanet/UMLS definitions; drug structures from PubChem
(7,105 of 7,957 drugs; biologics have none).

**Split.** Diseases with drug labels are divided into training (1,159), validation (155) and test (234) diseases. Diseases joined
by a direct MONDO link (parent, child or near-duplicate) always fall on the same side, so no test disease has such a
neighbour in training (`tests/test_data.py`). Test diseases contribute no drug edge to training.

## Performance (227 held-out test diseases, 2,196 drug–disease pairs)

| Model | MRR [95% CI] | Hits@1 | Hits@10 | Median rank | AUROC |
|---|---|---|---|---|---|
| Random | 0.003 | 0.000 | 0.004 | 1,363 | 0.504 |
| Popularity | 0.172 | 0.134 | 0.222 | 279 | 0.853 |
| Similar-disease kNN | 0.261 [0.192–0.331] | 0.222 | 0.315 | 97 | 0.877 |
| R-GCN + features, 3-seed ensemble | 0.284 [0.231–0.342] | 0.226 | 0.390 | 27 | 0.922 |
| **Deployed hybrid** | **0.319** [0.254–0.381] | **0.266** | **0.405** | 29 | **0.921** |

Paired bootstrap over diseases (5,000 draws): deployed hybrid vs kNN **+0.058 MRR (95% CI +0.036 to +0.082)**, Hits@10 +0.091;
pre-trained features in the hybrid +0.037 MRR (+0.018 to +0.056). Protein-sequence features (ESM-2) lowered validation MRR
and are not used.

**Rediscovery** (disease held out of training): sildenafil → erectile dysfunction #3; baricitinib → COVID-19 #20 of 2,755
(this link is not in PrimeKG); thalidomide → multiple myeloma #207; minoxidil → hair loss #1,292 (a miss: the effect was an
accidental side effect with no trace in the graph).

## Subgroups

| Test diseases | n | MRR | Hits@10 |
|---|---|---|---|
| Rare (Orphanet) | 87 | 0.498 | 0.601 |
| Common / not in Orphanet | 140 | 0.253 | 0.333 |
| ≤ 10 known disease genes | 156 | 0.213 | 0.325 |
| > 10 known disease genes | 71 | 0.427 | 0.486 |

The model is not worse for rare diseases. It is markedly worse for **diseases with few known genes**; the app flags diseases
with fewer than three genes as low-confidence.

## Ethical considerations and known failure modes

- **Hub bias.** Drugs that treat many diseases (corticosteroids) rank highly everywhere. The atlas offers a hub-corrected ranking and the Skeptic flags hub drugs.
- **Incomplete labels.** PrimeKG lists only known indications; a top candidate that is not a listed indication may be untested rather than wrong, and some true uses are missing.
- **Literature and trial evidence rediscover the known.** They inflate validation numbers and are reported separately; they favour well-studied diseases.
- **Direction of effect is not modelled by the graph.** A drug can be ranked highly for a disease it would worsen (an inhibitor of a gene whose loss causes the disease). A warning exists, but public direction data covers too few targets to validate it (`reports/direction_eval_curated.json`).
- **Blood-brain barrier** is estimated from structure (95.8% of nervous-system drugs pass the rule vs 57.3% of others), not measured.
- **FAERS** counts reflect reporting, not incidence.
- Docking uses rigid receptors and Vina scores; it is a sanity filter against decoys, not an affinity prediction.

## Caveats and recommendations

- Treat every output as a starting point for expert review and laboratory testing.
- Prefer candidates with independent support (a mechanistic path, expression reversal, docking above decoys) and no Skeptic objections.
- Re-run `scripts/run_training_eval.sh` after updating PrimeKG; check `pytest` passes (split leakage, metrics, leakage masking).
- PrimeKG has no timestamps, so this is not a prospective (time-split) evaluation.
