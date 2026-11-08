---
name: likelihood-array-postprocessing-threshold-and-power-scaling
description: Use when a ModiFinder per-atom likelihood array Ω has been computed and
  must be cleaned/sharpened prior to evaluation (e.g. Average-distance scoring) or
  visualization, to suppress low-confidence atoms and emphasize top candidates.
when_to_use_negative:
- When the raw Ω is needed for untransformed inspection of diffuse/ambiguous likelihoods,
  since thresholding and power-4 scaling deliberately discard low-score atoms and
  flatten spread
- When scoring spectra that have not passed ModiFinder's upstream steps (MS/MS alignment,
  40 ppm substructure assignment, peak filtering), since the raw array would not be
  a valid input
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ModiFinder
  role: Produces the raw per-atom modification-site likelihood array Ω that this post-processing
    transforms; available as command line tool and web interface
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:f861a7913e43a205bef3e7d8588a7b2e82897b959c4a68bb1ce957e7be26b74c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - ModiFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Likelihood array postprocessing: threshold and power scaling

## Summary

Applies ModiFinder's final post-processing to the per-atom modification-site likelihood array Ω: zeroing atom scores below 0.5, raising values to the power of 4, and renormalizing, to sharpen contrast among high-scoring candidate atoms before downstream evaluation.

## When to use

Use when a ModiFinder per-atom likelihood array Ω has been computed and must be cleaned/sharpened prior to evaluation (e.g. Average-distance scoring) or visualization, to suppress low-confidence atoms and emphasize top candidates.

## When NOT to use

- When the raw Ω is needed for untransformed inspection of diffuse/ambiguous likelihoods, since thresholding and power-4 scaling deliberately discard low-score atoms and flatten spread
- When scoring spectra that have not passed ModiFinder's upstream steps (MS/MS alignment, 40 ppm substructure assignment, peak filtering), since the raw array would not be a valid input

## Inputs

- Raw ModiFinder per-atom likelihood array Ω over the atoms of the known compound (e.g. Compound from GNPS accession CCMSLIB00010113829)
- 2D molecular graph of the known compound (atoms and bonds) for mapping atom scores

## Outputs

- Post-processed likelihood array Ω: values <0.5 zeroed, all values raised to the 4th power, renormalized to sum 1
- Per-atom intermediate contributions and computed scores written to evaluation_report.json

## How to apply

Take the raw likelihood array Ω over atoms of the known compound produced by ModiFinder's MS/MS alignment and combinatorial-fragmentation scoring. First normalize Ω so its values sum to 1. Then apply the hard threshold: any value below 0.5 is set to 0, which removes diffuse low-confidence atom assignments. To further highlight differences among the remaining high-scoring atoms, raise all values to the power of 4, then renormalize to sum to 1. The rationale is that the Average-distance evaluation rewards peaked, proximity-concentrated distributions over ambiguous ones, so thresholding plus power scaling directly amplifies the contrast the metric is sensitive to (illustrated in SI Fig. SI-5/SI-16). Persist the resulting array and per-atom contributions (e.g. to evaluation_report.json) so evaluation can be traced atom-by-atom.

## Related tools

- **ModiFinder** (Produces the raw per-atom modification-site likelihood array Ω that this post-processing transforms; available as command line tool and web interface) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
omega[omega < 0.5] = 0; omega = omega ** 4; omega /= omega.sum()  # ModiFinder final post-processing of likelihood scores
```

## Evaluation signals

- After thresholding, every nonzero atom score is ≥ 0.5 in the pre-power array
- After power-4 scaling and renormalization, the array sums to 1 (within floating-point tolerance)
- Sharpened arrays yield higher Average-distance scores for peaked, proximity-concentrated distributions than for diffuse ones, consistent with SI Fig. SI-5 examples
- evaluation_report.json contains both the final scores and intermediate per-atom contributions, allowing per-atom tracing
- Threshold (0.5) and exponent (4) match the article's specified values exactly

## Limitations

- Hard zeroing of values below 0.5 discards all sub-threshold atoms, so genuinely plausible but weakly scored sites become indistinguishable from impossible ones
- Power-of-4 scaling heavily concentrates mass on the single top atom; on ambiguous spectra this can mask real ambiguity that the Average-distance metric's ambiguity-cover term is meant to expose

## Evidence

- [readme] ModiFinder scope: [evidence span withheld — anchor status: not_found]
