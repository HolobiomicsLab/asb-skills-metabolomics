---
name: mass-accuracy-tolerance-filtering
description: Use when annotating IM-MS metabolite features (e.g. [M-H]- adducts) from
  a precursor exact mass and want to prune candidate structures using a mass window
  plus a ΔCCS threshold.
when_to_use_negative:
- Your training set does not support the adduct of interest — CCSP 2.0's current training
  sets do not cover dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source
  fragments
- You need filtering by retention time or tandem MS evidence rather than CCS — this
  skill relies on IM-MS CCS predictions and mass error only
- Your candidates span multiple IM-MS instrument types (e.g. travelling wave vs drift
  tube) where CCS threshold behavior has not yet been investigated
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Predicts CCS values for each mass-window candidate with a linear support vector
    regression model so that ΔCCS filtering can be applied
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred V1.2.0
  role: Computes the 1613 2D molecular descriptors consumed by the CCSP SVR model
    from RDKit-encoded MOL files
- name: RDKit
  role: Encodes candidate SMILES structures as MOL files prior to descriptor calculation
- name: PubChem (Entrez)
  role: Source of candidate annotations via the Entrez Global Query Cross-Database
    Search System at a 10 ppm exact-mass tolerance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:178970e408482c7bc9aabb568350fdbac7e1ad403f997bd9c8efe8a72dbfc75f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - Mordred V1.2.0
  - PubChem (Entrez)
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# mass-accuracy-tolerance-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds exact-mass candidate lists from PubChem at a defined mass tolerance (e.g. 10 ppm) and then filters those candidates by ΔCCS threshold using CCSP 2.0-predicted collision cross sections, retaining true annotations while removing false positives in ion-mobility metabolomics.

## When to use

Use when annotating IM-MS metabolite features (e.g. [M-H]- adducts) from a precursor exact mass and want to prune candidate structures using a mass window plus a ΔCCS threshold.

## When NOT to use

- Your training set does not support the adduct of interest — CCSP 2.0's current training sets do not cover dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments
- You need filtering by retention time or tandem MS evidence rather than CCS — this skill relies on IM-MS CCS predictions and mass error only
- Your candidates span multiple IM-MS instrument types (e.g. travelling wave vs drift tube) where CCS threshold behavior has not yet been investigated

## Inputs

- Observed precursor exact masses for deprotonated [M-H]- ions (e.g. 162 validation compounds from Supplemental Table 4)
- PubChem candidate lists retrieved via Entrez/Biopython at a 10 ppm mass tolerance
- Trained CCSP 2.0 linear-kernel SVR model (e.g. trained on 397 calibration compounds with 1613 Mordred descriptors from RDKit-encoded MOL files)

## Outputs

- Per-compound candidate lists annotated with predicted CCS values (598,208 candidates total in the validation; 15 to 18,399 per compound)
- Threshold-sweep table (filtering_stats.csv) recording % true annotations retained and median % candidates filtered at each ΔCCS threshold from 0% to 15%

## How to apply

For each feature, query PubChem via the Entrez Global Query Cross-Database Search System (Biopython) for candidates whose exact mass (computed with Mordred) falls within 10 ppm of the observed ion's exact mass. Encode each candidate's SMILES with RDKit, compute 1613 2D descriptors with Mordred V1.2.0, and predict CCS with the trained CCSP 2.0 linear-kernel SVR model. Sweep the ΔCCS filtering threshold (0% to 15%; 2.8–3% is the working range) and record the percentage of true annotations retained versus the median percentage of candidates filtered; a 3% threshold retained 76.5% of true annotations and filtered a median 47.3% of candidates at a 10 ppm window (24.9% for same-formula candidates). Restricting candidate lists to the true compound's chemical formula before filtering reduces the filtered fraction, so report both all-candidate and same-formula statistics.

## Related tools

- **CCSP 2.0** (Predicts CCS values for each mass-window candidate with a linear support vector regression model so that ΔCCS filtering can be applied) — https://github.com/facundof2016/CCSP2.0
- **Mordred V1.2.0** (Computes the 1613 2D molecular descriptors consumed by the CCSP SVR model from RDKit-encoded MOL files)
- **RDKit** (Encodes candidate SMILES structures as MOL files prior to descriptor calculation)
- **PubChem (Entrez)** (Source of candidate annotations via the Entrez Global Query Cross-Database Search System at a 10 ppm exact-mass tolerance)

## Examples

```
python -c "from Bio import Entrez; Entrez.esearch(db='pccompound', term='10.0:11.0[mass]')"; jupyter notebook "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"
```

## Evaluation signals

- At ΔCCS 3% and 10 ppm, the filtering_stats.csv output must show 76.5% of true annotations retained
- Median % of candidates filtered at ΔCCS 3% with 10 ppm window must be 47.3%; the same-formula-restricted run must give 24.9%
- The combined candidate count across the 162 compounds must total 598,208 (15 to 18,399 per compound), excluding 9 compounds without exact PubChem matches
- Per-candidate CCS predictions should fall within the CCSP 2.0 median relative error benchmarks (1.25% for [M-H]-, 1.73% for [M+H]+, 1.87% for [M+Na]+)
- Threshold sweep must be monotone in retained fraction: higher ΔCCS thresholds retain ≥ the true annotations retained at lower thresholds

## Limitations

- Current training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments
- CCS predictions across multiple IM-MS instrument types (drift tube, travelling wave) not yet explored, so threshold behavior may differ by platform
- Effect of experimental sources of variance on the choice of CCS threshold not yet investigated
- Requires Python familiarity, limiting accessibility; Google Colab is limited to 12-hour continuous runs and disconnects after ~20 minutes idle

## Evidence

- [readme] Collision Cross Section Predictor 2.0 is an open source Python notebook intended to help ion-mobility scientists predict collision cross sections with user-curated training sets.: [evidence span withheld — anchor status: external_doc]
