---
name: collision-energy-resolved-msms-comparison
description: Use when you have isomer-pair MS/MS spectra acquired at multiple collision
  energies and need sensitivity/specificity of conventional single-spectrum similarity
  matching per NCE condition as a baseline.
when_to_use_negative:
- The goal is longitudinal (multi-CE fragment-profile) matching rather than establishing
  the cross-sectional baseline
- Only a single collision energy per compound is available, so no per-NCE comparison
  across conditions is possible
- Spectra are not isomer pairs (e.g., diverse candidate-vs-reference matching in an
  untargeted identification run) — use the metabolite-identification notebooks instead
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzbatch.py
  role: Library of commonly used methods for mass-spectrometry data processing (file
    operations, data validation, generation, calculation, extraction) reused for similarity
    computation
- name: 001-007.ipynb
  role: Notebook series containing isomer similarity calculations executed in sequence
    to reproduce the benchmark
- name: Jaccard similarity algorithm
  role: Optimized similarity algorithm evaluated alongside dot product, modified dot
    product, and entropy in the cross-sectional comparison
- name: conda
  role: Creates the longprof environment with Python 3.8.13 for reproducible execution
- name: pip
  role: Installs pinned dependencies from requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:bffec5f7117986840c1fddc71801a0217802740f675e1434288acaa988528807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# collision-energy-resolved-msms-comparison

## Summary

Benchmarks cross-sectional MS/MS similarity algorithms (dot product, modified dot product, entropy, optimized Jaccard) across 13 individual collision energies and stepped NCE 20/40/60% on isomer pairs, to establish the best-case baseline that longitudinal fragment profiling must beat.

## When to use

Use when you have isomer-pair MS/MS spectra acquired at multiple collision energies and need sensitivity/specificity of conventional single-spectrum similarity matching per NCE condition as a baseline.

## When NOT to use

- The goal is longitudinal (multi-CE fragment-profile) matching rather than establishing the cross-sectional baseline
- Only a single collision energy per compound is available, so no per-NCE comparison across conditions is possible
- Spectra are not isomer pairs (e.g., diverse candidate-vs-reference matching in an untargeted identification run) — use the metabolite-identification notebooks instead

## Inputs

- CQMD isomer-pair MS/MS spectra (processed through upstream CQMU database-construction steps)
- Spectra acquired at 13 individual NCEs and stepped NCE 20/40/60%
- Isomer vs. non-isomer pair labels for binary sensitivity/specificity calculation

## Outputs

- cross_sectional_similarity_metrics.csv with full algorithm × NCE sensitivity/specificity table
- Reference-condition metrics (e.g., NCE 70%: 93.25% sensitivity / 87.96% specificity; stepped NCE: 92.25% / 77.38%)
- Per-mode extrema such as entropy CP-mode (100% / 50.83%) and dot product CP-mode (99.80% / 83.77%)

## How to apply

Set up the Python 3.8.13 environment (conda create -n longprof; pip install -r requirements.txt) and reuse the mzbatch.py library plus the 001-007.ipynb notebooks from the LongitudinalProf-MSMS-Method repository. For every CQMD isomer pair, compute cross-sectional similarity scores with four algorithms — dot product, modified dot product, entropy, and the optimized Jaccard similarity algorithm — at each of the 13 individual NCEs and at stepped NCE 20/40/60%. Binarize scores at the 0.95 threshold and compute sensitivity and specificity per algorithm × NCE condition over isomer/non-isomer pairs. The rationale is to quantify the ceiling of conventional single-spectrum matching before applying longitudinal fragment profiles, which treat predominant and minute fragments independently across CEs.

## Related tools

- **mzbatch.py** (Library of commonly used methods for mass-spectrometry data processing (file operations, data validation, generation, calculation, extraction) reused for similarity computation)
- **001-007.ipynb** (Notebook series containing isomer similarity calculations executed in sequence to reproduce the benchmark)
- **Jaccard similarity algorithm** (Optimized similarity algorithm evaluated alongside dot product, modified dot product, and entropy in the cross-sectional comparison)
- **conda** (Creates the longprof environment with Python 3.8.13 for reproducible execution)
- **pip** (Installs pinned dependencies from requirements.txt)

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt  # then run 001-007.ipynb cells in Jupyter Notebook to compute isomer similarity per NCE
```

## Evaluation signals

- Recomputed NCE 70% metrics must match reported sensitivity 93.25% and specificity 87.96% within numeric tolerance
- Stepped NCE 20/40/60% must reproduce 92.25% sensitivity and 77.38% specificity
- Entropy CP-mode must reproduce 100% sensitivity / 50.83% specificity; dot product CP-mode must reproduce 99.80% / 83.77%
- cross_sectional_similarity_metrics.csv must contain one row per algorithm × NCE condition (13 individual NCEs + stepped NCE)
- All values must be computed after binarizing similarity scores at the 0.95 threshold

## Limitations

- Both cross-sectional best cases (NCE 70% and stepped NCE) fall short of longitudinal profiling (98.89% / 92.66%), so this benchmark alone underestimates achievable performance
- The CQMD in-house database is not publicly available (no accession given), limiting out-of-the-box reproduction
- Stepped NCE 20/40/60% is inferior to the optimal single NCE 70% in this cross-sectional setting

## Evidence

- [results] Cross-sectional evaluation of CQMD isomer pairs performed best at NCE 70% with a 0.95 threshold, achieving an average sensitivity of 93.25% and a specificity of 87.96% across four modes: [evidence span withheld — anchor status: not_found]
- [results] For every isomer pair, compute cross-sectional similarity scores with four algorithms — dot product, modified dot product, entropy, and the optimized Jaccard similarity algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] The Jaccard algorithm, rather than the classic dot product or optimal entropy algorithm used in the cross-sectional evaluation, was optimized: "The Jaccard algorithm, rather than the classic dot product or optimal entropy algorithm used in the cross-sectional evaluation, was optimized"
- [readme] Run every cell of "001-007.ipynb" in sequence in Jupyter Notebook.: "Run every cell of "001-007.ipynb" in sequence in Jupyter Notebook. Make sure that the data set is stored in the same path as specified in the script."
- [readme] mzbatch.py: This file contains a library of commonly used methods for mass spectrometry data processing: "mzbatch.py: This file contains a library of commonly used methods for mass spectrometry data processing, offering modules for file operations, data validation, data generation, data calculation, and"
