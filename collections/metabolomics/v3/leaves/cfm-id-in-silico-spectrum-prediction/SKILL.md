---
name: cfm-id-in-silico-spectrum-prediction
description: 'Use when benchmarking modification-site-localization on analog pairs:
  candidate analog structures are available (e.g. via substructure permutation), each
  has a known formula, and an empirical MS2 spectrum of the unknown compound is available
  for cosine scoring.'
when_to_use_negative:
- When no analog structure or modification substructure can be derived from the unknown
  compound — CFM-ID needs candidate structures to predict spectra for
- When the MS2-unknown spectrum is not available or peak preprocessing (intensity
  filtering below 1% of base peak, normalization to Euclidean norm 1) cannot be applied
  first
- When the goal is ranking candidate molecular formulas rather than localizing a modification
  site on a known parent structure
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID
  role: Docker container used to predict in-silico MS/MS spectra for each candidate
    analog structure with prob_thresh 0.001 and [M+H]+ trained models
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ModiFinder
  role: Benchmarking framework in which CFM-ID predictions are scored against MF-R/MF-O
    and random baselines using the Average-distance evaluation
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:c96fbae1dac2bd95327063be10494d6ed9873f9cc2dc51d72ecc345552add6c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cfm-id-in-silico-spectrum-prediction

## Summary

Use the CFM-ID docker container to predict MS/MS spectra for candidate analog structures (e.g. permuted modification-site variants), then rank them by cosine similarity against the unknown compound's MS2 spectrum. In the ModiFinder benchmark this serves as a competitive baseline for modification-site localization.

## When to use

Use when benchmarking modification-site-localization on analog pairs: candidate analog structures are available (e.g. via substructure permutation), each has a known formula, and an empirical MS2 spectrum of the unknown compound is available for cosine scoring.

## When NOT to use

- When no analog structure or modification substructure can be derived from the unknown compound — CFM-ID needs candidate structures to predict spectra for
- When the MS2-unknown spectrum is not available or peak preprocessing (intensity filtering below 1% of base peak, normalization to Euclidean norm 1) cannot be applied first
- When the goal is ranking candidate molecular formulas rather than localizing a modification site on a known parent structure

## Inputs

- candidate analog molecular structures (e.g. S-known with the modification substructure permuted across atom sites, with bond-type matching first then fallback)
- MS2-unknown spectrum of the unannotated compound
- precursor/adduct specification ([M+H]+ in the benchmark)
- CFM-ID docker container (hub.docker.com/r/wishartlab/cfmid)

## Outputs

- predicted MS/MS spectrum per candidate analog structure
- cosine similarity score per candidate vs. MS2-unknown
- per-atom likelihood distribution for the modification site (via Average-distance evaluation)

## How to apply

Generate each candidate analog (for example by permuting the modification substructure across S-known with matching then fallback bond types), then run the CFM-ID docker container (hub.docker.com/r/wishartlab/cfmid) with prob_thresh 0.001 and [M+H]+ trained models to predict in-silico MS/MS spectra for every candidate. Score each candidate's predicted spectrum against the MS2-unknown spectrum by cosine similarity and use the similarity as the candidate's likelihood of being the true analog. Apply the paper's Average-distance evaluation — an exponential decay over 2D graph distance normalized by graph diameter — to the resulting per-atom likelihood scores, and compare them against Random Choice, Random Distribution, and ModiFinder's MF-R/MF-O baselines.

## Related tools

- **CFM-ID** (Docker container used to predict in-silico MS/MS spectra for each candidate analog structure with prob_thresh 0.001 and [M+H]+ trained models) — https://hub.docker.com/r/wishartlab/cfmid
- **ModiFinder** (Benchmarking framework in which CFM-ID predictions are scored against MF-R/MF-O and random baselines using the Average-distance evaluation) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
docker run wishartlab/cfmid cfm-id --prob_thresh 0.001 --adduct [M+H]+ --input candidate_analog.sdf --output predicted_msms.mgf
```

## Evaluation signals

- Each candidate analog's predicted MS/MS spectrum should be compared against MS2-unknown via cosine similarity — check the similarity falls within [0,1]
- Per-candidate likelihoods should map onto per-atom scores on S-known and be evaluable with the Average-distance metric (exponential decay over 2D graph distance normalized by graph diameter)
- In the ModiFinder benchmark, CFM-ID's Average-distance score is expected to be lower than both Random Choice and Random Distribution on average (+0.266 average delta for MF-R over CFM-ID), so verify the CFM-ID baseline reproduces that ordering
- Check the CFM-ID run succeeded by confirming each candidate produced a non-empty predicted spectrum at prob_thresh 0.001 with [M+H]+ models

## Limitations

- CFM-ID scored worse than Random Choice and Random Distribution baselines on the ModiFinder benchmark, so it should not be assumed to outperform simpler approaches on this task
- Requires a candidate analog structure per permutation — expensive when many candidate sites must be tested
- Accuracy depends on quality of the MS2-unknown preprocessing (intensity filtering, normalization) and on the [M+H]+ trained models; other adducts are not validated here

## Evidence

- [methods] CFM-ID docker run parameters: [evidence span withheld — anchor status: not_found]
- [results] Random baseline comparison: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder description: [evidence span withheld — anchor status: not_found]
