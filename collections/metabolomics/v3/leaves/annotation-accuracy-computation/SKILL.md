---
name: annotation-accuracy-computation
description: Use when top-ranked molecular formula annotations (e.g. from SIRIUS or
  ZODIAC) for untargeted LC-MS/MS compounds must be scored against a manually validated
  ground truth across datasets.
when_to_use_negative:
- The input is raw mzML/mzXML spectra or preprocessing output — run OpenMS and SIRIUS/ZODIAC
  annotation first; this skill only scores already-produced top-ranked formula assignments
- The goal is de novo molecular formula annotation or structure (CSI:FingerID) search
  rather than measuring annotation accuracy against a ground truth
- No manually validated or spectral-library-search ground truth exists for the compounds,
  so 'correct' cannot be defined per the article's evaluation protocol
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: SIRIUS
  role: Produces the baseline top-ranked molecular formula annotations that are scored
    against the ground truth; also used to compute fragmentation trees for MS/MS spectra
    of the monoisotopic peak
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Reranks the top 50 SIRIUS molecular formula candidates using Bayesian statistics
    via Gibbs sampling (optionally integrating library-hit anchors); its top-ranked
    assignments are scored against ground truth to quantify error reduction
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS
  role: 'Preprocessing: the modified OpenMS 2.4.0 processes input mzML/mzXML files
    and discards low-quality MS/MS spectra before annotation, ensuring only high-quality
    spectra enter the benchmark'
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:31d907d10d2c7c0bb0668838d7843f32b2e67d60aee32d1b69c3bc2d6c6ba20e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# annotation-accuracy-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for quantifying molecular-formula annotation accuracy: parse top-ranked formula assignments from SIRIUS and ZODIAC for each compound, match them against a ground truth established by spectral library search plus manual validation, and report per-dataset accuracy and incorrect-annotation rates.

## When to use

Use when top-ranked molecular formula annotations (e.g. from SIRIUS or ZODIAC) for untargeted LC-MS/MS compounds must be scored against a manually validated ground truth across datasets.

## When NOT to use

- The input is raw mzML/mzXML spectra or preprocessing output — run OpenMS and SIRIUS/ZODIAC annotation first; this skill only scores already-produced top-ranked formula assignments
- The goal is de novo molecular formula annotation or structure (CSI:FingerID) search rather than measuring annotation accuracy against a ground truth
- No manually validated or spectral-library-search ground truth exists for the compounds, so 'correct' cannot be defined per the article's evaluation protocol

## Inputs

- Deposited SIRIUS molecular formula annotation results per dataset (from https://bio.informatik.uni-jena.de/data/)
- Deposited ZODIAC molecular formula annotation results per dataset (same repository, incl. anchor and no-anchor runs)
- Ground-truth molecular formula files established by spectral library search and manual validation
- Dataset identifiers/accessions: dendroides (MSV000080502), NIST1950 (MSV000081364), tomato, diatoms, mice stool
- Optional: number of evaluable ground truth compounds per dataset (e.g. 197 for dendroides)

## Outputs

- Per-dataset, per-method annotation accuracy (fraction of correct top-ranked molecular formulas)
- Per-dataset, per-method incorrect-annotation rate
- Error-reduction factor (ratio of incorrect-annotation rates, e.g. ~16.17-fold on dendroides)
- 5 datasets x 2 methods benchmark CSV table with accuracy and error-rate columns

## How to apply

Download the deposited SIRIUS and ZODIAC annotation results and the ground-truth formula files from https://bio.informatik.uni-jena.de/data/. Parse each annotation file to extract the top-ranked molecular formula SIRIUS assigns (by enumerating formula candidates) and the top-ranked formula ZODIAC assigns (ZODIAC reranks the top 50 SIRIUS candidates via Gibbs sampling, integrating spectral library search hits as anchors). Match each top-ranked formula against the ground truth established by spectral library search and manual validation for each of the five datasets (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool). Compute per-dataset annotation accuracy as the fraction of correct top-ranked formulas and the corresponding incorrect-annotation rate for each method; for dendroides (197 evaluable ground truth compounds) derive the error-reduction factor as the ratio of SIRIUS to ZODIAC incorrect rates (reported 16.2-fold, 50.76% vs 96.95% correct). Write the 5 datasets x 2 methods table with accuracy and error-rate columns to CSV, and validate that computed values reproduce the published figures before reuse.

## Related tools

- **SIRIUS** (Produces the baseline top-ranked molecular formula annotations that are scored against the ground truth; also used to compute fragmentation trees for MS/MS spectra of the monoisotopic peak) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Reranks the top 50 SIRIUS molecular formula candidates using Bayesian statistics via Gibbs sampling (optionally integrating library-hit anchors); its top-ranked assignments are scored against ground truth to quantify error reduction) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS** (Preprocessing: the modified OpenMS 2.4.0 processes input mzML/mzXML files and discards low-quality MS/MS spectra before annotation, ensuring only high-quality spectra enter the benchmark) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
python compute_annotation_accuracy.py --dataset dendroides --sirius-results sirius_results.tsv --zodiac-results zodiac_results.tsv --ground-truth ground_truth.tsv --out benchmark.csv
```

## Evaluation signals

- Computed accuracies reproduce the published per-dataset values: dendroides 50.76% (SIRIUS) vs 96.95% (ZODIAC); NIST1950 8.51% vs 5.32%; tomato 4.44% vs 2.22%; diatoms ~12.90% roughly halved; mice stool 30.23% vs 20.93%
- Derived dendroides error-reduction factor ≈16.17-fold, matching the reported 16.2-fold decrease in incorrect annotations
- Number of evaluable compounds matches the ground truth (e.g. 197 for dendroides) and every compound's top-ranked formula is checked in both SIRIUS and ZODIAC outputs
- Output CSV has exactly 5 datasets x 2 methods rows with accuracy and error-rate columns populated
- Where anchors are used, verify via 10-fold cross-validation that anchor integration reflects the reported small effect on molecular formula annotations

## Limitations

- Reproducing the paper's setup requires the modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; high-quality adduct annotations must be established during preprocessing, or they confound formula scoring
- ZODIAC's Gibbs-sampling run uses ten parallel chains, a burn-in of 1,000 epochs and sampling of 2,000 epochs — deviating from these settings may change the sampled rankings and hence accuracy
- A complete evaluation of novel molecular formulas (e.g. C24H47BrNO8P) would require full structural elucidation, which is experimentally infeasible; ground truth covers only library-supported compounds
- SIRIUS 4's formula annotation performance drops substantially for compounds above 500 Da, so accuracy estimates for high-mass datasets (e.g. dendroides, most compounds >700 Da) are the most method-sensitive

## Evidence

- [results] On the dendroides dataset (197 evaluable ground truth compounds), SIRIUS correctly annotated 50.76% of molecular formulas, while ZODIAC without anchors correctly annotated 96.95%: [evidence span withheld — anchor status: not_found]
- [methods] ZODIAC reranks the top 50 SIRIUS candidates via Gibbs sampling: "ZODIAC builds upon SIRIUS and uses, say, the top 50 molecular formula annotations from SIRIUS as candidates for one compound"
- [methods] Evaluation against ground truth established by spectral library search and manual validation: "We evaluated SIRIUS and ZODIAC against a ground truth which was established by spectral library search and manual validation"
- [discussion] Anchor contribution to accuracy: [evidence span withheld — anchor status: not_found]
- [discussion] Reproducibility requires modified OpenMS: [evidence span withheld — anchor status: not_found]
- [methods] Gibbs-sampling run parameters: "We used ten parallel chain, a burn-in of 1,000 epochs and sampling of 2,000 epochs"
- [intro] SIRIUS performance limitation for high-mass compounds: "even SIRIUS has problems annotating molecular formula for compounds above 500 Da"
- [discussion] Benchmark datasets and accessions: "dendroides (MSV000080502)"
