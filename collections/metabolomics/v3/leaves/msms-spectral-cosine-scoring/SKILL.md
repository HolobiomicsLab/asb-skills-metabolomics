---
name: msms-spectral-cosine-scoring
description: Use when ranking candidate metabolite annotations from predicted vs.
  experimental tandem mass spectra, or when judging whether the top-ranked annotation
  is trustworthy via the Δcos gap between top-1 and top-2 candidates.
when_to_use_negative:
- When experimental spectra are already available in a curated reference library and
  a direct library match suffices — the paper notes SingleFrag is intended to overcome
  the lack of comprehensive reference spectral libraries, not to replace curated empirical
  matching.
- When the question is peak-intensity prediction rather than presence/absence — SingleFrag
  deliberately focuses on predicting the presence or absence of each peak, and intensity-based
  scoring is out of scope for this skill.
- When spectra were acquired at a single collision energy without multi-collision-energy
  context — the paper warns that public libraries (GNPS, MSDial, MassBank) often contain
  only one or a few collision energies per compound, limiting the reliability of cosine-based
  ranking against such spectra.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SingleFrag
  role: Generates in silico MS/MS spectra (predicted peak presence at individual MZ
    positions) that are scored against experimental spectra by cosine similarity for
    metabolite annotation and ranking.
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Produces the 300-dimensional molecular embedding that is fed into the ANN
    models to drive per-fragment peak presence predictions used in the scoring pipeline.
- name: CFM-ID
  role: Benchmark comparator tool whose predicted spectra are scored with the same
    cosine-similarity and precision/recall metrics to validate SingleFrag's ranking
    performance.
- name: 3DMolMS
  role: Benchmark comparator tool whose predicted spectra are scored with the same
    cosine-similarity and precision/recall metrics to validate SingleFrag's ranking
    performance.
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:7928758732b54d3f29e54718fe83a81603b1cf0c75c562ede35643de52433870
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - CFM-ID
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msms-spectral-cosine-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Scores in silico predicted MS/MS spectra against experimental spectra and ranks candidate metabolites by cosine similarity, using the cosine-similarity gap between the top and second candidate (Δcos > 0.2) as a reliability signal for top-ranked annotations.

## When to use

Use when ranking candidate metabolite annotations from predicted vs. experimental tandem mass spectra, or when judging whether the top-ranked annotation is trustworthy via the Δcos gap between top-1 and top-2 candidates.

## When NOT to use

- When experimental spectra are already available in a curated reference library and a direct library match suffices — the paper notes SingleFrag is intended to overcome the lack of comprehensive reference spectral libraries, not to replace curated empirical matching.
- When the question is peak-intensity prediction rather than presence/absence — SingleFrag deliberately focuses on predicting the presence or absence of each peak, and intensity-based scoring is out of scope for this skill.
- When spectra were acquired at a single collision energy without multi-collision-energy context — the paper warns that public libraries (GNPS, MSDial, MassBank) often contain only one or a few collision energies per compound, limiting the reliability of cosine-based ranking against such spectra.

## Inputs

- Ranked candidate lists from the upstream annotation run, retaining per-query cosine scores for the top two candidates and top-1 correctness labels
- Predicted MS/MS spectra from SingleFrag (ANN, GNN, COM models) reconstructed on a 0.2 m/z grid, plus experimental MS/MS spectra from the test compounds (2,929 compounds from the SingleFrag Figshare dataset inputMols file)
- Per-MZ-position prediction files (results/dfPredictionPeak_*.pickle) and reconstructed spectra (results/predSpectra.json)

## Outputs

- Per-tool, per-molecule table of precision, recall, accuracy, F1, and cosine similarity (fig3g-i_tool_metrics.csv)
- Δcos-gap-stratified top-1 correctness figure (gap_stratified_accuracy.png) and per-bin accuracy table (gap_stratified_accuracy.csv)

## How to apply

Reconstruct predicted spectra by running the trained ANN, GNN, and COM models over the 1,000 most frequented MZ positions and applying the per-position classification thresholds (thresholdsANN/GNN/COM.pickle) to call peak presence, then discretize both predicted and experimental spectra on a common 0.2 m/z binning grid. For each tool and molecule, compute precision, recall, accuracy, F1, and cosine similarity of the predicted spectrum versus the experimental spectrum to rank candidates. To assess annotation reliability, compute Δcos = cos(top-1) − cos(top-2) for each query spectrum, bin queries by Δcos (with the 0.2 gap threshold highlighted as the key cutoff), and report top-1 correctness in each bin. Rationale: the paper's Fig. 4d-f stratification shows that when the cosine-similarity gap between the top and second ranked candidates exceeds 0.2, the top candidate is the correct molecule in the majority of cases.

## Related tools

- **SingleFrag** (Generates in silico MS/MS spectra (predicted peak presence at individual MZ positions) that are scored against experimental spectra by cosine similarity for metabolite annotation and ranking.) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Produces the 300-dimensional molecular embedding that is fed into the ANN models to drive per-fragment peak presence predictions used in the scoring pipeline.)
- **CFM-ID** (Benchmark comparator tool whose predicted spectra are scored with the same cosine-similarity and precision/recall metrics to validate SingleFrag's ranking performance.)
- **3DMolMS** (Benchmark comparator tool whose predicted spectra are scored with the same cosine-similarity and precision/recall metrics to validate SingleFrag's ranking performance.) — https://github.com/JosieHong/3DMolMS

## Examples

```
import numpy as np; delta_cos = cos_top1 - cos_top2; bins = pd.cut(delta_cos, bins=10); acc = df.groupby(bins)['top1_correct'].mean()
```

## Evaluation signals

- Bins with Δcos exceeding 0.2 must show top-candidate correctness above 0.5 (top candidate correct in the majority of cases), consistent with the signal reported in Fig. 4d-f.
- All three SingleFrag variants (ANN, GNN, COM) must achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS on the 2,929-compound test set at 0.2 m/z binning, with recall comparable to (slightly better than) 3DMolMS for ANN and COM.
- Reconstructed spectra must be consistent between the per-MZ-position prediction files (dfPredictionPeak_*.pickle) and the reconstructed spectra file (predSpectra.json).
- The cosine-similarity distribution of predicted vs. experimental spectra must be computed on a common 0.2 m/z discretization grid shared across all tools and molecules.

## Limitations

- The GNN model shows slightly lower performance than the ANN model; the authors attribute this to deliberately simple, unoptimized graph features and layer architectures rather than a fundamental limitation of graph neural networks.
- The 0.2 Δcos cutoff is an empirical stratification threshold from Fig. 4d-f; correctness in the majority of cases does not guarantee correctness for any individual query.
- Public spectral libraries (GNPS, MSDial, MassBank) often contain spectra at only one or a few collision energies, which can make incomplete or less informative reference spectra a poor target for cosine-based ranking.

## Evidence

- [results] Stratifying annotation correctness by Δcos shows that when the difference in cosine similarity scores between the top and second ranked candidates exceeds 0.2, the top candidate is the correct molecule in the majority of cases.: [evidence span withheld — anchor status: not_found]
- [results] all three SingleFrag variants achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS, with recall comparable to 3DMolMS (slightly better for ANN and combined models).: [evidence span withheld — anchor status: not_found]
