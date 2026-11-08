---
name: candidate-filter-statistics-evaluation
description: Use when evaluating whether a learned scoring/filtering model retains
  the true chemical formula among its top-ranked candidates (e.g., top-256 recovery)
  and how large enumerated candidate sets are before filtering.
when_to_use_negative:
- 'Input spectra are negative-mode: MIST-CF/FastFilter models were trained only for
  positive-mode data, so filtered rankings would be out of scope.'
- Candidate lists are already database-restricted (e.g., NIST/GNPS/HMDB/PubChem lookup);
  the skill targets large database-independent RDBE-permissive enumerations from SIRIUS
  decomp.
- Goal is final structure annotation rather than formula candidate filtering statistics;
  use CSI:FingerID or downstream MIST-CF evaluation instead.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: FastFilter
  role: Light-weight neural network energy model that scores and ranks each candidate
    formula against its MS/MS spectrum; supplies the top-256 filtered ranking being
    evaluated.
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Its dynamic-programming 'decomp' module enumerates the RDBE-plausible formula
    candidates whose pre-filter counts and post-filter recovery are measured.
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:0f87dca4faac275884b4d5f6ddb49655cee59163382459c22093559ff2dbea73
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - FastFilter
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# candidate-filter-statistics-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies how well a neural formula-filtering model (FastFilter, from the MIST-CF pipeline) downselects large RDBE-permissive chemical formula candidate sets by measuring pre-filter candidate-count distributions and the top-k recovery rate of the true formula.

## When to use

Use when evaluating whether a learned scoring/filtering model retains the true chemical formula among its top-ranked candidates (e.g., top-256 recovery) and how large enumerated candidate sets are before filtering.

## When NOT to use

- Input spectra are negative-mode: MIST-CF/FastFilter models were trained only for positive-mode data, so filtered rankings would be out of scope.
- Candidate lists are already database-restricted (e.g., NIST/GNPS/HMDB/PubChem lookup); the skill targets large database-independent RDBE-permissive enumerations from SIRIUS decomp.
- Goal is final structure annotation rather than formula candidate filtering statistics; use CSI:FingerID or downstream MIST-CF evaluation instead.

## Inputs

- MS/MS training spectra from NPLIB1 and NIST20 datasets
- Enumerated RDBE-permissive formula candidate sets produced by SIRIUS decomp
- Pretrained FastFilter checkpoint (trained via src/mist_cf/fast_form_score/train.py with configs/fast_filter.yaml)

## Outputs

- Per-spectrum ranked candidate formula list (top-256)
- Pre-filter candidate-count distribution across spectra
- Top-256 true-formula recovery rate
- Summary table fast_filter_stats.csv

## How to apply

Enumerate RDBE-permissible formula candidates for each MS1 precursor mass using SIRIUS's dynamic-programming 'decomp' module, then load the pretrained FastFilter checkpoint trained per src/mist_cf/fast_form_score/train.py with settings from configs/fast_filter.yaml. Run src/mist_cf/fast_form_score/predict.py to score every candidate formula against its MS/MS spectrum with the FastFilter energy model and rank candidates per spectrum. Compute the pre-filter candidate-count distribution (the paper reports over 15% of training spectra from NPLIB1 and NIST20 have more than 5,000 candidates) and the top-256 recovery rate of the true formula (99% in the paper). Evaluate the rankings with run_scripts/fast_filter/launch_fast_pred.py and write the summary table to fast_filter_stats.csv.

## Related tools

- **FastFilter** (Light-weight neural network energy model that scores and ranks each candidate formula against its MS/MS spectrum; supplies the top-256 filtered ranking being evaluated.) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Its dynamic-programming 'decomp' module enumerates the RDBE-plausible formula candidates whose pre-filter counts and post-filter recovery are measured.) — https://github.com/boecker-lab/sirius

## Examples

```
python run_scripts/fast_filter/launch_fast_pred.py
```

## Evaluation signals

- Top-256 recovery rate of the true formula should reach approximately 99% on NPLIB1/NIST20 training spectra, matching the paper's reported figure.
- Pre-filter candidate-count distribution should show that a non-trivial fraction of spectra (over 15% in the paper) exceed 5,000 RDBE-permissive candidates, confirming filtering is necessary.
- fast_filter_stats.csv is produced and contains per-dataset summary statistics rather than an empty or malformed table.
- Every input spectrum receives a ranked candidate list where the true formula, when present in the decomp enumeration, is retained within the top 256.
- Ranking quality should be cross-checked with run_scripts/fast_filter/launch_fast_pred.py evaluation output.

## Limitations

- Recovery statistics depend on SIRIUS decomp enumerations; the pipeline still relies upon SIRIUS's algorithmic decomposition of exact masses into formula candidates.
- FastFilter and the surrounding MIST-CF models were trained only for positive-mode data, so these statistics do not transfer to negative ion mode.
- The pretrained public model (NPLIB1/CANOPUS) may be less performant than NIST20-trained models, particularly for Orbitrap or higher-resolution data.

## Evidence

- [results] Over 15% of training spectra (NPLIB1 and NIST20) have more than 5,000 RDBE-permissive formula candidates, and FastFilter recovers the true formula 99% of the time within its top 256 candidates, keeping the pipeline tractable.: [evidence span withheld — anchor status: not_found]
