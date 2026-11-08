---
name: roc-curve-auc-analysis
description: Use when you have per-compound ranked candidate structure lists with
  a known correct structure and want to quantify whether a score (e.g. COSMIC confidence
  score or tool hit score) separates correct from incorrect annotations.
when_to_use_negative:
- Input is raw LC-MS/MS data (mzML/mzXML) rather than per-compound ranked hit scores
  with known correct structures — process spectra first with SIRIUS 4/CSI:FingerID.
- You need to estimate a score threshold for a new biological dataset — the article
  warns that accurate FDR estimation from small-molecule fragmentation spectra remains
  highly challenging and thresholds are practical guidance only.
- You want to interpret the confidence score as the probability that an annotation
  is correct — the article states this is impossible by design of the score.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: COSMIC
  role: Confidence-score workflow whose scores are evaluated with ROC/AUC and FDR-level
    hit counts against contest-method baselines
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Structure candidate ranking whose scores are compared against COSMIC confidence
    scores in the ROC evaluation
  repo: https://github.com/sirius-ms/sirius
- name: MetFrag
  role: In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC
    comparison
- name: MAGMa+
  role: In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC
    comparison
- name: CFM-ID
  role: In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC
    comparison
- name: CSI:FingerID IOKR
  role: In silico tool variant whose CASMI 2016 hit scores serve as a baseline in
    the ROC/AUC comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:4c6a21b3c84b773d49b513b9667f83eda88197fca565ef791b9cdfbcdbb9a426
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# roc-curve-auc-analysis

## Summary

Evaluates how well a ranking score (e.g. COSMIC confidence scores versus in silico tool hit scores from MetFrag, MAGMa+, CFM-ID, CSI:FingerID) separates correct from incorrect structure annotations on CASMI 2016, using ROC curves, AUC, hop plots, and correct-hit counts at fixed FDR levels.

## When to use

Use when you have per-compound ranked candidate structure lists with a known correct structure and want to quantify whether a score (e.g. COSMIC confidence score or tool hit score) separates correct from incorrect annotations.

## When NOT to use

- Input is raw LC-MS/MS data (mzML/mzXML) rather than per-compound ranked hit scores with known correct structures — process spectra first with SIRIUS 4/CSI:FingerID.
- You need to estimate a score threshold for a new biological dataset — the article warns that accurate FDR estimation from small-molecule fragmentation spectra remains highly challenging and thresholds are practical guidance only.
- You want to interpret the confidence score as the probability that an annotation is correct — the article states this is impossible by design of the score.

## Inputs

- Per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR from the CASMI 2016 results web page
- COSMIC confidence scores for the 127 positive-ion Orbitrap standard compounds
- Known correct structure per CASMI 2016 query compound

## Outputs

- casmi2016_roc_hop_results.csv with AUC, correct-hit counts at 10% and 0% FDR, and mean ranks per method
- ROC curves exported as PNG figures
- Hop plots exported as PNG figures

## How to apply

Download the published per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR from the CASMI 2016 results web page, and load the COSMIC confidence scores computed for the 127 positive-ion Orbitrap standard compounds. For each score, rank candidate structures per compound against the known correct structure and compute ROC curves and hop plots per method. Estimate the area under the ROC curve per score (contest-method baselines expected at 0.40-0.55 versus COSMIC at ~0.82) and count correct hits at 10% and 0% FDR (57 with the biomolecule structure database over 123 queries; 16 with ChemSpider over 127 queries). Also compute the average rank of correct hits per method to quantify rank improvement (56.5 to 33.9). Write all metrics (AUC, correct-hit counts at both FDR levels, mean ranks) to a results CSV and export ROC/hop plots as figures.

## Related tools

- **COSMIC** (Confidence-score workflow whose scores are evaluated with ROC/AUC and FDR-level hit counts against contest-method baselines) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Structure candidate ranking whose scores are compared against COSMIC confidence scores in the ROC evaluation) — https://github.com/sirius-ms/sirius
- **MetFrag** (In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC comparison)
- **MAGMa+** (In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC comparison)
- **CFM-ID** (In silico tool whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC comparison)
- **CSI:FingerID IOKR** (In silico tool variant whose CASMI 2016 hit scores serve as a baseline in the ROC/AUC comparison)

## Examples

```
python casmi2016_roc_hop.py --tool-scores casmi2016_hits.csv --cosmic-scores cosmic_127_scores.csv --correct-structures casmi2016_truth.csv --output casmi2016_roc_hop_results.csv --plots png
```

## Evaluation signals

- COSMIC AUC (~0.82) exceeds the contest-method baselines expected at 0.40-0.55
- Correct hits at 10% FDR: 57 when searching the biomolecule structure database (123 queries); at 0% FDR: 16 when searching ChemSpider (127 queries)
- Mean rank of correct hits improves from 56.5 to 33.9 for COSMIC confidence scores
- Results CSV contains AUC, correct-hit counts at both FDR levels, and mean ranks for every method evaluated
- ROC and hop plots are exported as PNG figures for each method

## Limitations

- Previous in silico methods cannot distinguish between correct and incorrect annotations, so only the confidence-score methodology — not raw hit scores — achieves useful separation
- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- Accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging; a threshold such as 0.64 is practical guidance only and no guarantee in either direction
- Evaluations rely on reference measurements with known structures; no ground truth exists for biological datasets

## Evidence

- [abstract] Baseline separation quality of in silico scores: "Previous in silico methods search in structure databases but cannot distinguish between correct and incorrect annotations"
- [discussion] Comparison-scores provenance: "Scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR were downloaded from the CASMI 2016 results web page"
- [abstract] Expected outcome of the ROC evaluation: "COSMIC annotates a substantial number of hits at low false discovery rates and outperforms spectral library search"
- [discussion] FDR-threshold caveat for score-based selection: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
