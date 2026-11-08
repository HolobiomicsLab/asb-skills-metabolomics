---
name: annotation-accuracy-benchmarking
description: Use when SIRIUS/ZODIAC result files are available for LC-MS/MS datasets
  with known ground-truth formulas, and you need to quantify how annotation correctness
  varies with ZODIAC score thresholds (e.g. 0.7–0.99).
when_to_use_negative:
- You only have raw mzML/mzXML spectra that have not yet been processed by OpenMS
  preprocessing and SIRIUS/ZODIAC — the score-threshold benchmark requires the ZODIAC
  marginal output files
- You want to benchmark structure (CSI:FingerID) identification rather than molecular
  formula annotation — this skill evaluates only the formula-annotation step
- You have no ground-truth formulas (no spectral library hits or manual validation),
  so correctness cannot be established
tools:
- name: ZODIAC
  role: Source of the posterior (marginal) scores per compound's top-ranked molecular
    formula candidate that are thresholded for the accuracy benchmark
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: SIRIUS
  role: Provides the top-50 molecular formula candidates per compound that ZODIAC
    reranks; used to compute fragmentation trees for each MS/MS spectrum
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS 2.4.0 (modified)
  role: Preprocessing of input mzML/mzXML files (discarding low-quality MS/MS spectra)
    prior to SIRIUS/ZODIAC runs used in the benchmark
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:9041de87fc16d6f74aed93935ea0c36b085f0862e2174c3cfde43aec7db1ca10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# annotation-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark molecular formula annotation accuracy by comparing ZODIAC posterior scores against ground-truth formulas established via spectral library search and manual validation, sweeping score thresholds to quantify the correctness/retention trade-off.

## When to use

Use when SIRIUS/ZODIAC result files are available for LC-MS/MS datasets with known ground-truth formulas, and you need to quantify how annotation correctness varies with ZODIAC score thresholds (e.g. 0.7–0.99).

## When NOT to use

- You only have raw mzML/mzXML spectra that have not yet been processed by OpenMS preprocessing and SIRIUS/ZODIAC — the score-threshold benchmark requires the ZODIAC marginal output files
- You want to benchmark structure (CSI:FingerID) identification rather than molecular formula annotation — this skill evaluates only the formula-annotation step
- You have no ground-truth formulas (no spectral library hits or manual validation), so correctness cannot be established

## Inputs

- SIRIUS/ZODIAC result files (deposited at https://bio.informatik.uni-jena.de/data/) containing ZODIAC marginal scores per compound
- Ground-truth molecular formula assignments established by spectral library search (e.g. NIST17) and manual validation
- Dataset metadata for the five evaluated datasets: NIST1950, dendroides, tomato, diatoms, mice stool

## Outputs

- zodiac_threshold_accuracy.csv with one row per score threshold, giving fraction correct and fraction of compounds retained
- Per-dataset correctness/retention comparison across thresholds

## How to apply

Download the deposited SIRIUS and ZODIAC result files from https://bio.informatik.uni-jena.de/data/ and parse the ZODIAC marginal (posterior) score for each compound's top-ranked molecular formula candidate. Load ground-truth molecular formulas established by spectral library search (e.g. against the NIST17 library) and manual validation, and match each compound's ZODIAC annotation against its ground truth. For threshold 0.9 and neighbors (0.7, 0.8, 0.95, 0.99), compute the fraction of correct annotations among compounds scoring above each threshold and the fraction of compounds retained per dataset. Write the threshold-vs-accuracy table to zodiac_threshold_accuracy.csv with one row per threshold. The rationale is that ZODIAC reranks SIRIUS candidates via Bayesian statistics with Gibbs sampling, so its posterior score should stratify annotation reliability; the paper reports >96.5% correctness at threshold 0.9 while retaining 52.05%–88.24% of compounds per dataset.

## Related tools

- **ZODIAC** (Source of the posterior (marginal) scores per compound's top-ranked molecular formula candidate that are thresholded for the accuracy benchmark) — https://bio.informatik.uni-jena.de/software/sirius/
- **SIRIUS** (Provides the top-50 molecular formula candidates per compound that ZODIAC reranks; used to compute fragmentation trees for each MS/MS spectrum) — https://github.com/sirius-ms/sirius
- **OpenMS 2.4.0 (modified)** (Preprocessing of input mzML/mzXML files (discarding low-quality MS/MS spectra) prior to SIRIUS/ZODIAC runs used in the benchmark) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('zodiac_threshold_accuracy.csv'); assert t.set_index('threshold').loc[0.9,'accuracy']>0.965"
```

## Evaluation signals

- Accuracy at the 0.9 threshold must exceed 96.5% correct molecular formula annotations, matching the reported value
- Retention at threshold 0.9 must fall within the reported 52.05%–88.24% range across the datasets
- Accuracy should increase monotonically as the threshold rises from 0.7 toward 0.99/0.999 (compare with the paper's filter levels: score ≥ 0.99 plus ≥ 90% explained intensity, and score ≥ 0.999 plus ≥ 95% explained intensity)
- Output CSV must contain one row per threshold tested with both accuracy and compound-retention columns populated for every dataset
- Cross-check that every compound classified as correct has a ZODIAC annotation formula identical to its library-search/manual-validation ground-truth formula

## Limitations

- Ground truth depends on spectral library search plus manual validation, so coverage of ground-truth formulas limits which compounds can be evaluated
- Library-hit anchors contribute only a small effect on molecular formula annotations, so accuracy measured on anchored compounds may not fully transfer
- A complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible
- Reproducibility requires the modified OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- ZODIAC's parameters (ten parallel chains, burn-in of 1,000 epochs, sampling of 2,000 epochs on 40 cores) affect scores and thus threshold behavior

## Evidence

- [results] ground truth via library search: "We evaluated SIRIUS and ZODIAC against a ground truth which was established by spectral library search and manual validation"
- [results] filter thresholds: "Filtering less restrictively (ZODIAC score at least 0.99, at least 90 % of the MS/MS spectrum intensity being explained by SIRIUS)"
- [results] strict filter level: "a) minimum ZODIAC score of 0.999, b) at least 95 % of the MS/MS spectrum intensity being explained by SIRIUS"
- [discussion] data availability: "SIRIUS and ZODIAC results and a virtual machine to reproduce the data are available from https://bio.informatik.uni-jena.de/data/."
- [readme] SIRIUS software scope: [evidence span withheld — anchor status: external_doc]
