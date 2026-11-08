---
name: roc-threshold-sensitivity-specificity-evaluation
description: Use when benchmarking spectral similarity algorithms on isomer vs non-isomer
  pairs across multiple collision energies (13 individual NCEs or stepped NCE 20/40/60%)
  and you need threshold-based sensitivity/specificity figures to set identification
  criteria.
when_to_use_negative:
- The spectra were acquired at only a single collision energy, so neither the 13-individual-NCE
  sweep nor stepped NCE 20/40/60% comparison can be reproduced
- There is no isomer/non-isomer ground-truth labeling of pairs, making sensitivity/specificity
  and ROC analysis undefined
- The results are intended for clinical decision-making — the README states the data
  'are intended for experimental reference only and should not be directly interpreted
  or used to guide human medicine'
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0091
tools:
- name: 001-007.ipynb
  role: Notebook series running spectrum extraction, spectrum filtering, invalid fragment
    removal, fragment rationality assessment, CQMU database construction, and isomer
    similarity calculations that feed the sensitivity/specificity evaluation
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Python library of mass-spectrometry data-processing methods (file operations,
    data validation, generation, calculation, extraction) reused while computing isomer
    similarity scores
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jupyter Notebook
  role: Execution environment in which every cell of 001-007.ipynb is run in sequence
    to reproduce the metric tables
- name: Python 3.8.13
  role: Pinned runtime environment (conda env 'longprof') with dependencies installed
    from requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:f90ba49ab8115171608490c60768875bbb4dff6af70e9071491d352b64cf0659
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# roc-threshold-sensitivity-specificity-evaluation

## Summary

Establishes MS/MS spectral-matching identification criteria via ROC analysis over thousands of isomer pairs in the CQMD database, then evaluates each similarity algorithm (dot product, modified dot product, entropy, optimized Jaccard) x collision-energy condition as binary sensitivity/specificity at a 0.95 score threshold.

## When to use

Use when benchmarking spectral similarity algorithms on isomer vs non-isomer pairs across multiple collision energies (13 individual NCEs or stepped NCE 20/40/60%) and you need threshold-based sensitivity/specificity figures to set identification criteria.

## When NOT to use

- The spectra were acquired at only a single collision energy, so neither the 13-individual-NCE sweep nor stepped NCE 20/40/60% comparison can be reproduced
- There is no isomer/non-isomer ground-truth labeling of pairs, making sensitivity/specificity and ROC analysis undefined
- The results are intended for clinical decision-making — the README states the data 'are intended for experimental reference only and should not be directly interpreted or used to guide human medicine'

## Inputs

- CQMD isomer-pair MS/MS spectra processed through the CQMU database-construction pipeline (spectrum extraction, filtering, invalid fragment removal, fragment rationality assessment)
- Similarity scores computed at 13 individual NCEs and stepped NCE 20/40/60% using dot product, modified dot product, entropy, and optimized Jaccard algorithms
- Isomer/non-isomer pair labels for each CQMD isomer pair

## Outputs

- cross_sectional_similarity_metrics.csv: full algorithm x NCE sensitivity/specificity table binarized at the 0.95 threshold
- ROC-based identification criteria for the longitudinal profiling method
- Benchmark comparison against reference conditions (e.g., NCE 70%: 93.25%/87.96%; stepped NCE: 92.25%/77.38%; longitudinal: 98.89%/92.66%)

## How to apply

Load the CQMD isomer-pair MS/MS spectra (built through the CQMU database-construction steps in 001-007.ipynb with mzbatch.py). For every isomer pair, compute cross-sectional similarity scores with four algorithms — dot product, modified dot product, entropy, and the optimized Jaccard similarity algorithm — at each of the 13 individual NCEs and at stepped NCE 20/40/60%. Binarize scores at the 0.95 threshold and compute sensitivity and specificity per algorithm x NCE condition over the isomer/non-isomer pairs; identification criteria themselves are established by ROC analysis on the thousands of isomers in the in-house CQMD database. Reference conditions to anchor against: NCE 70% (sensitivity 93.25%, specificity 87.96%), stepped NCE 20/40/60% (92.25%/77.38%), entropy CP-mode (100%/50.83%), dot product CP-mode (99.80%/83.77%), and longitudinal profiling (98.89%/92.66%). Write the full algorithm x NCE metric table to cross_sectional_similarity_metrics.csv for comparison against these reported values.

## Related tools

- **001-007.ipynb** (Notebook series running spectrum extraction, spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction, and isomer similarity calculations that feed the sensitivity/specificity evaluation) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Python library of mass-spectrometry data-processing methods (file operations, data validation, generation, calculation, extraction) reused while computing isomer similarity scores) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jupyter Notebook** (Execution environment in which every cell of 001-007.ipynb is run in sequence to reproduce the metric tables)
- **Python 3.8.13** (Pinned runtime environment (conda env 'longprof') with dependencies installed from requirements.txt)

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Recomputed NCE 70% condition matches sensitivity 93.25% and specificity 87.96% within numeric tolerance
- Recomputed stepped NCE 20/40/60% condition matches 92.25% sensitivity and 77.38% specificity
- Recomputed entropy CP-mode reaches 100%/50.83% and dot product CP-mode reaches 99.80%/83.77%
- Longitudinal profiling reference values of 98.89% sensitivity / 92.66% specificity are recovered and exceed all cross-sectional conditions
- The output table cross_sectional_similarity_metrics.csv exists and contains one sensitivity/specificity row per algorithm x NCE condition

## Limitations

- The CQMD in-house database is not publicly available (no accession or public access stated), limiting independent reproduction of the isomer-pair benchmark
- Cross-sectional evaluation, even at its best condition (NCE 70%, 93.25%/87.96%), falls short of longitudinal profiling (98.89%/92.66%), so single-CE threshold tuning has an inherent accuracy ceiling
- Stepped NCE 20/40/60% is inferior to the optimal single NCE 70% for cross-sectional evaluation, so stepped acquisition does not rescue single-spectrum matching
- The README disclaims liability: data are for experimental reference only and not to guide human medicine

## Evidence

- [supplementary] Binarize the scores at the 0.95 threshold and compute sensitivity and specificity per algorithm x NCE condition: [evidence span withheld — anchor status: not_found]
