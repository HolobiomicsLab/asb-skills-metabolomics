---
name: hop-plot-visualization
description: Use when you have per-compound ranked hit scores from structure annotation
  tools plus a known correct structure per query, and you need ROC/hop-plot curves
  and AUC to judge score separation quality.
when_to_use_negative:
- No known correct structure is available per query (e.g. biological datasets without
  ground truth, where evaluations must rely on reference measurements)
- Input is a list of raw spectra rather than ranked candidate structure scores per
  compound
edam_operation: http://edamontology.org/operation_3439
tools:
- name: COSMIC
  role: Confidence scores whose separation performance (AUC ~0.82, 57 correct hits
    at FDR <10%) is compared against contest-method baselines in the ROC/hop-plot
    evaluation
  repo: https://github.com/sirius-ms/sirius
- name: MetFrag
  role: Baseline in silico hit scores downloaded from the CASMI 2016 results page
    and plotted on ROC/hop plots as a contest-method comparison (AUC 0.40-0.55 expected)
- name: CSI:FingerID
  role: Baseline hit scores (original and IOKR variants) downloaded from the CASMI
    2016 results page and evaluated in the ROC/hop-plot comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:ea4c37375b37bbb3c5894eb4cdf2c6abfc0bf073c9cecb7c72c5cf1a1a1fffef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CFM-ID
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID IOKR
  - MAGMa+
  - MetFrag
  edam_operation: http://edamontology.org/operation_3439
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# hop-plot-visualization

## Summary

Rank structure candidates per compound against the known correct structure to compute ROC curves and hop plots that quantify how well an annotation score separates correct from incorrect hits, as done when evaluating COSMIC's confidence score against CASMI 2016 in silico tool scores.

## When to use

Use when you have per-compound ranked hit scores from structure annotation tools plus a known correct structure per query, and you need ROC/hop-plot curves and AUC to judge score separation quality.

## When NOT to use

- No known correct structure is available per query (e.g. biological datasets without ground truth, where evaluations must rely on reference measurements)
- Input is a list of raw spectra rather than ranked candidate structure scores per compound

## Inputs

- Per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR downloaded from the CASMI 2016 results web page
- COSMIC confidence scores computed for the 127 positive-ion Orbitrap standard compounds
- Known correct structure labels per CASMI 2016 query compound

## Outputs

- casmi2016_roc_hop_results.csv with per-method AUC, correct-hit counts at 10% and 0% FDR, and mean ranks of correct hits
- ROC curves exported as PNG figures
- Hop plots exported as PNG figures

## How to apply

Download the published per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR from the CASMI 2016 results web page, and load the COSMIC confidence scores computed for the 127 positive-ion Orbitrap standard compounds. For each hit score, rank the candidate structures per compound against the known correct structure and compute ROC curves and hop plots, treating the known structure as the positive class. Estimate the area under the ROC curve for each score and compare against expected contest-method baselines of 0.40-0.55 versus COSMIC at 0.82. Count correct hits at 10% and 0% FDR (57 with the biomolecule structure database on 123 queries; 16 with ChemSpider on 127 queries) and compute the average rank of correct hits per method to quantify the rank improvement from 56.5 to 33.9. Write all metrics (AUC, correct-hit counts at both FDR levels, mean ranks) to casmi2016_roc_hop_results.csv and export the ROC and hop plots as PNG figures.

## Related tools

- **COSMIC** (Confidence scores whose separation performance (AUC ~0.82, 57 correct hits at FDR <10%) is compared against contest-method baselines in the ROC/hop-plot evaluation) — https://github.com/sirius-ms/sirius
- **MetFrag** (Baseline in silico hit scores downloaded from the CASMI 2016 results page and plotted on ROC/hop plots as a contest-method comparison (AUC 0.40-0.55 expected))
- **CSI:FingerID** (Baseline hit scores (original and IOKR variants) downloaded from the CASMI 2016 results page and evaluated in the ROC/hop-plot comparison)

## Examples

```
from sklearn.metrics import roc_curve, auc; fpr, tpr, _ = roc(y_true, scores); print(auc(fpr, tpr))
```

## Evaluation signals

- COSMIC's AUC (~0.82) exceeds the contest-method baselines (0.40-0.55)
- Correct-hit counts at 10%/0% FDR reproduce the reported values: 57 hits with FDR below 10% searching the biomolecule structure database (123 queries) and 16 hits with FDR 0% searching ChemSpider (127 queries)
- Mean rank of correct hits per method shows the reported rank improvement from 56.5 to 33.9
- casmi2016_roc_hop_results.csv contains AUC, correct-hit counts at both FDR levels, and mean ranks, and ROC/hop figures are exported as PNG

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets
- A score threshold such as 0.64 might serve as practical guidance but is clearly no guarantee in either direction

## Evidence

- [other] Rank candidates and compute curves: [evidence span withheld — anchor status: not_found]
- [other] Expected AUC comparison: [evidence span withheld — anchor status: not_found]
- [discussion] Score interpretation caveat: [evidence span withheld — anchor status: not_found]
- [discussion] Ground truth limitation: "Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets."
