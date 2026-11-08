---
name: retention-time-zscore-annotation-ranking
description: Use when annotating small molecules with only m/z and retention times
  (no MS/MS spectra), and you need to rank a mass-filtered candidate list using RT
  predictions projected to your chromatographic method from ~10 identified molecules.
when_to_use_negative:
- When fragmentation (MS/MS) spectra are available for the unknowns — spectral matching
  is the stronger evidence and this skill targets the spectra-unavailable case
- When the target chromatographic method has no identified molecules to train the
  projection (the method needs as few as 10, but at least some identified RTs)
- When mass filtering alone returns three or fewer candidates — the evaluation and
  ranking are only meaningful for queries with more than three candidates
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0154
tools:
- name: cmmrt
  role: Python package implementing the RT predictor, GP meta-learning projections,
    and the annotation ranking experiment
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 2,214 fingerprints (MACCS166, ECFP, PFP) and 5,666 descriptors
    used to train the DNN RT predictor
- name: Python
  role: Runs the cmmrt pipeline (Makefile rules train_projections / test_projections)
    and the ranking evaluation script
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:6bae307ad22ca3beeeb5a0154801aeb6959a024a474ae9c183ca8b10b2d9932c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# retention-time-zscore-annotation-ranking

## Summary

Ranks metabolite annotation candidates by combining accurate-mass filtering (10 ppm tolerance) with retention time z-scores computed from DNN RT predictions projected to the target chromatographic method via a meta-learned GP, so the correct molecule lands in the top three in ~68% of queries.

## When to use

Use when annotating small molecules with only m/z and retention times (no MS/MS spectra), and you need to rank a mass-filtered candidate list using RT predictions projected to your chromatographic method from ~10 identified molecules.

## When NOT to use

- When fragmentation (MS/MS) spectra are available for the unknowns — spectral matching is the stronger evidence and this skill targets the spectra-unavailable case
- When the target chromatographic method has no identified molecules to train the projection (the method needs as few as 10, but at least some identified RTs)
- When mass filtering alone returns three or fewer candidates — the evaluation and ranking are only meaningful for queries with more than three candidates

## Inputs

- DNN (fingerprint-based) RT predictions for the 6,823 KEGG-numbered HMDB molecules (annotation database)
- Experimental RTs of ~10-50 identified molecules of the target chromatographic method (FEM long, FEM orbitrap plasma, LIFE old, or RIKEN from PredRet)
- Leave-one-CM-out meta-trained GP projection (constant mean, degree-4 polynomial kernel)
- Measured accurate masses with 10 ppm tolerance and Gaussian noise (sigma = 10/3 ppm, truncated to +/-10 ppm)

## Outputs

- annotation_ranking_accuracy.csv with top-three accuracies (with standard errors) and mass-only baseline values per chromatographic method and training-set size
- Per-query RT z-scores z = |y* - mu(x*)| / sigma(x*) ranking mass-filtered candidates

## How to apply

Build a simulated annotation database of DNN (fingerprint-based) RT predictions for the 6,823 KEGG-numbered HMDB molecules, then project predicted RTs to each target chromatographic method (FEM long, FEM orbitrap plasma, LIFE old, RIKEN) using the leave-one-CM-out meta-trained GP (constant mean, degree-4 polynomial kernel). For each target-set molecule, simulate a 10 ppm accurate-mass search by adding Gaussian noise (zero mean, sigma = 10/3 ppm, truncated to +/-10 ppm) to the true mass so the correct molecule is always a candidate. Rank returned candidates by the RT z-score z = |y* - mu(x*)| / sigma(x*), where mu and sigma are the GP posterior mean and standard deviation of the projection. Restrict evaluation to mass queries returning more than three candidates, compute the percentage of queries where the true molecule is in the top three for 10-50 training points, and compare against a mass-only baseline with random tie-breaking.

## Related tools

- **cmmrt** (Python package implementing the RT predictor, GP meta-learning projections, and the annotation ranking experiment) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 2,214 fingerprints (MACCS166, ECFP, PFP) and 5,666 descriptors used to train the DNN RT predictor)
- **Python** (Runs the cmmrt pipeline (Makefile rules train_projections / test_projections) and the ranking evaluation script) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make test_projections  # then evaluate ranking: python cmmrt/annotation/rank_candidates.py --cm FEM_long --train_size 10 --ppm 10 --out annotation_ranking_accuracy.csv
```

## Evaluation signals

- Global mean top-three accuracy reaches approximately 68% with 10 training points and approximately 70% with 50 training points
- RT z-score ranking exceeds the mass-only random-tie-break baseline per chromatographic method (e.g., 59.57% for FEM long), consistent with Table 5 of the paper
- Every simulated 10 ppm mass query returns the true molecule as a candidate (noise truncated to +/-10 ppm)
- Ranking is restricted to queries returning more than three candidates; accuracies are reported with standard errors per CM and training-set size in annotation_ranking_accuracy.csv

## Limitations

- Annotations rely only on m/z ratios and RTs because fragmentation spectra are often unavailable in pilot untargeted studies; performance degrades when RT projections are uncertain (few identified molecules)
- The simulated mass search guarantees the correct molecule is a candidate; real mass searches may miss it or return many more candidates
- For CEU Mass Mediator integration the code is under continued development; only the `paper` branch reproduces the paper's results

## Evidence

- [abstract] our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [intro] Molecules are filtered using the m/z ratio and the predicted RTs of those molecules are then projected to experimental RTs: [evidence span withheld — anchor status: not_found]
- [abstract] a novel Bayesian meta-learning approach that can learn from just a few molecules ... it is enough that just as few as 10 molecules of a given experiment have been identified: [evidence span withheld — anchor status: not_found]
- [methods] RT predictions ... for the 6,823 molecules with KEGG number in the Human Metabolome DataBase (HMDB): [evidence span withheld — anchor status: not_found]
- [readme] Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN: "Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN. For each system, 
the GP is meta-trained using all other CMs from the PredRet database"
