---
name: zodiac-posterior-score-threshold-analysis
description: Use when deposited or locally generated SIRIUS/ZODIAC result files are
  available and the goal is to pick a ZODIAC posterior-score cutoff for high-confidence
  molecular formula annotation.
when_to_use_negative:
- The goal is structure identification rather than molecular formula annotation confidence
  — structure database search on novel formulas will always return incorrect hits
  that often go unnoticed
- Only raw SIRIUS isotope-pattern/fragmentation-tree output exists and ZODIAC posterior
  scores have not been computed, so there is no score to threshold
- The task is adduct or fragment annotation — the article notes adduct information
  cannot be reliably deduced from isotope pattern and MS/MS data and should be established
  during preprocessing instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ZODIAC
  role: Source of the marginal (posterior) score per compound's top-ranked molecular
    formula candidate that is swept across thresholds
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: SIRIUS
  role: Produces the top-50 molecular formula candidates that ZODIAC reranks; the
    fraction of MS/MS spectrum intensity explained by SIRIUS can be combined with
    score filters
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Preprocessing of input mzML/mzXML files and discarding low-quality MS/MS spectra;
    reproducibility requires a modified OpenMS 2.4.0 plus a patch file
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:ae2af59d88479588b077d59cf8cb0a32d36408e25186296a2823b54d829322dd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# zodiac-posterior-score-threshold-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sweep over ZODIAC marginal (posterior) score thresholds that quantifies the fraction of correct molecular formula annotations and the fraction of compounds retained per threshold, using ground-truth formulas established by spectral library search and manual validation.

## When to use

Use when deposited or locally generated SIRIUS/ZODIAC result files are available and the goal is to pick a ZODIAC posterior-score cutoff for high-confidence molecular formula annotation.

## When NOT to use

- The goal is structure identification rather than molecular formula annotation confidence — structure database search on novel formulas will always return incorrect hits that often go unnoticed
- Only raw SIRIUS isotope-pattern/fragmentation-tree output exists and ZODIAC posterior scores have not been computed, so there is no score to threshold
- The task is adduct or fragment annotation — the article notes adduct information cannot be reliably deduced from isotope pattern and MS/MS data and should be established during preprocessing instead

## Inputs

- SIRIUS/ZODIAC result files containing the ZODIAC marginal (posterior) score for each compound's top-ranked molecular formula candidate
- Ground-truth molecular formulas established by spectral library search and manual validation
- Preprocessed LC-MS/MS input (mzML/mzXML processed with OpenMS, low-quality MS/MS spectra discarded)

## Outputs

- zodiac_threshold_accuracy.csv with one row per score threshold, giving the fraction of correct annotations and the fraction of compounds retained
- Threshold-vs-accuracy comparison across thresholds such as 0.7, 0.8, 0.9, 0.95, 0.99

## How to apply

Download the deposited SIRIUS and ZODIAC result files from https://bio.informatik.uni-jena.de/data/, then parse the ZODIAC marginal (posterior) score assigned to each compound's top-ranked molecular formula candidate. Match each compound's ZODIAC annotation against its ground-truth molecular formula, established by spectral library search and manual validation. For threshold 0.9 and neighboring thresholds (e.g. 0.7, 0.8, 0.95, 0.99), compute the fraction of correct annotations among compounds whose ZODIAC score meets each threshold, along with the fraction of compounds retained per dataset. Write the threshold-vs-accuracy table to zodiac_threshold_accuracy.csv with one row per threshold. Rationale: the article shows filtering at a minimum ZODIAC score (e.g. 0.999 for novel formulas, 0.99 less restrictively) drives annotation confidence, and at threshold 0.9 accuracy exceeds 96.5% while keeping 52.05%–88.24% of compounds per dataset.

## Related tools

- **ZODIAC** (Source of the marginal (posterior) score per compound's top-ranked molecular formula candidate that is swept across thresholds) — https://bio.informatik.uni-jena.de/software/sirius/
- **SIRIUS** (Produces the top-50 molecular formula candidates that ZODIAC reranks; the fraction of MS/MS spectrum intensity explained by SIRIUS can be combined with score filters) — https://github.com/sirius-ms/sirius
- **OpenMS** (Preprocessing of input mzML/mzXML files and discarding low-quality MS/MS spectra; reproducibility requires a modified OpenMS 2.4.0 plus a patch file) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
python scripts/zodiac_threshold_accuracy.py --zodiac-results ./deposited/zodiac --ground-truth ./ground_truth_formulas.csv --thresholds 0.7,0.8,0.9,0.95,0.99 --output zodiac_threshold_accuracy.csv
```

## Evaluation signals

- Accuracy among compounds with ZODIAC score >= 0.9 must exceed 96.5% correct annotations, matching the reported value
- Coverage at threshold 0.9 must fall within the reported 52.05%–88.24% of compounds retained per dataset
- The threshold-vs-accuracy table should be monotone: accuracy non-decreasing and fraction of compounds retained non-increasing as the threshold rises from 0.7 to 0.99
- Ground-truth matching must reproduce the article's evaluation scheme — ground truth established by spectral library search and manual validation
- Recomputed scores/annotations should agree with the deposited SIRIUS and ZODIAC result files from https://bio.informatik.uni-jena.de/data/

## Limitations

- Ground truth is restricted to compounds matchable by spectral library search; complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible
- Cosine score cannot substitute for the ZODIAC score threshold: no distinct difference was found between the cosine score of ZODIAC's correct and incorrect molecular formula assignments
- Reproducing the pipeline from raw data requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- Even at high thresholds only a fraction of detected compounds is retained, so high-accuracy filtering trades coverage for confidence

## Evidence
