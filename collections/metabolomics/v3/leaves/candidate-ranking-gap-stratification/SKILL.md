---
name: candidate-ranking-gap-stratification
description: 'Decision support for judging how trustworthy a top-1 metabolite annotation
  is: after ranking candidate molecules by cosine similarity between predicted and
  experimental MS/MS spectra, stratify correctness by the Δcos gap to identify confidence
  cutoffs.'
when_to_use_negative:
- The upstream annotation run produced only a single ranked candidate per query (no
  second-best cosine score), since Δcos cannot be computed
- Spectra were matched against empirical rather than in silico-predicted spectra without
  cosine scores, so no ranking-gap signal exists
- Correctness labels are unavailable, making per-bin top-1 accuracy impossible to
  estimate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SingleFrag
  role: Generates the in silico MS/MS spectra used to rank candidate molecules by
    cosine similarity against experimental spectra, producing the top-1/top-2 score
    pairs this skill stratifies
  repo: github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds input molecule SMILES into the 300-dimensional vectors that feed SingleFrag's
    ANN models during upstream candidate annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:816b3d8b988b7f9a01b7721522823ad9444b451df966c1d53c1e43ebb14d5fd8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# candidate-ranking-gap-stratification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies annotation reliability by binning queries according to the cosine-similarity gap (Δcos) between the top-1 and top-2 candidate spectra predicted by SingleFrag and measuring top-1 correctness per bin. When Δcos exceeds 0.2, the top-ranked candidate is the correct molecule in the majority of cases (Fig. 4d-f).

## When to use

Decision support for judging how trustworthy a top-1 metabolite annotation is: after ranking candidate molecules by cosine similarity between predicted and experimental MS/MS spectra, stratify correctness by the Δcos gap to identify confidence cutoffs.

## When NOT to use

- The upstream annotation run produced only a single ranked candidate per query (no second-best cosine score), since Δcos cannot be computed
- Spectra were matched against empirical rather than in silico-predicted spectra without cosine scores, so no ranking-gap signal exists
- Correctness labels are unavailable, making per-bin top-1 accuracy impossible to estimate

## Inputs

- Ranked candidate annotation lists with per-query cosine scores for the top-1 and top-2 candidates
- Top-1 correctness labels (is the top-ranked candidate the true metabolite)
- Query-level table loadable as a pandas DataFrame

## Outputs

- Per-bin accuracy table of top-1 correctness versus Δcos bin (gap_stratified_accuracy.csv)
- Stratified reliability plot mirroring Fig. 4d-f (gap_stratified_accuracy.png)
- A Δcos threshold (0.2) usable as an annotation confidence cutoff

## How to apply

From the upstream annotation run, retain for each query spectrum the cosine scores of the top-1 and top-2 candidates plus a correctness label for the top-1 hit. Compute Δcos = cos(top-1) − cos(top-2) with numpy, bin the queries into equal-width bins spanning the score range using pandas, and highlight the 0.2 gap threshold — the key cutoff reported in the paper. In each bin compute the fraction of queries whose top-ranked candidate is correct, then plot correctness versus Δcos bin with matplotlib, mirroring the panel layout of Fig. 4d-f. Bins with Δcos > 0.2 should show top-1 correctness above 0.5, meaning the top candidate is the true metabolite in the majority of cases; use this threshold as a confidence criterion when deciding whether to accept an annotation or escalate the query to manual curation.

## Related tools

- **SingleFrag** (Generates the in silico MS/MS spectra used to rank candidate molecules by cosine similarity against experimental spectra, producing the top-1/top-2 score pairs this skill stratifies) — github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds input molecule SMILES into the 300-dimensional vectors that feed SingleFrag's ANN models during upstream candidate annotation)

## Examples

```
df['dcos'] = df['cos_top1'] - df['cos_top2']; df.groupby(pd.cut(df['dcos'], bins=np.arange(0, 1.0, 0.05))).accuracy.mean().plot(); df.to_csv('gap_stratified_accuracy.csv')
```

## Evaluation signals

- Bins with Δcos exceeding 0.2 must show top-1 correctness above 0.5 (top candidate correct in the majority of cases), consistent with Fig. 4d-f
- The per-bin accuracy table (gap_stratified_accuracy.csv) must contain one correctness fraction per Δcos bin that sums to the overall query count when weighted by bin counts
- Every query must have a non-negative Δcos since cos(top-1) ≥ cos(top-2) in a ranked list
- The monotonic trend of correctness versus Δcos should be inspected: higher gap bins should not show systematically lower correctness

## Limitations

- The 0.2 Δcos cutoff yields top-1 correctness only in the majority of cases, not certainty — annotations above the threshold can still be wrong and may need manual confirmation of molecular structures and fragmentation patterns
- Reliability depends on the quality of the upstream in silico spectra; SingleFrag's GNN model shows slightly lower performance than the ANN model, so score gaps from weaker models may stratify less cleanly
- The cutoff is empirical, derived from the paper's annotation experiments on the in silico library of nearly 2 million molecules, and should be re-validated on the specific instrument/platform of the user's data

## Evidence

- [other] Stratifying annotation correctness by Δcos shows that when the difference in cosine similarity scores between the top and second ranked candidates exceeds 0.2, the top candidate is the correct molecule in the majority of cases.: [evidence span withheld — anchor status: not_found]
- [other] Reliability of the top-ranked candidate as a function of the delta-cosine gap: [evidence span withheld — anchor status: not_found]
