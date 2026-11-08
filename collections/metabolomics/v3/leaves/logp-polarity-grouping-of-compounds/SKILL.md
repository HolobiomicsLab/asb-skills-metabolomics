---
name: logp-polarity-grouping-of-compounds
description: Use when evaluating formula-prediction output on a held-out test set
  and you need per-compound-property breakdowns (LogP polarity, mass range, chemical
  superclass) of top-k accuracy rather than a single pooled score.
when_to_use_negative:
- Spectra come from sources outside the FIDDLE test-compound space (e.g., molecules
  with atoms beyond C, H, O, N, F, S, Cl, P, B, I, Br, Na, and K, or m/z above 1500),
  where the annotation join and accuracy comparison are out of scope
- The goal is overall benchmark comparison against SIRIUS or BUDDY rather than property-stratified
  error analysis — pooled top-1/top-5 accuracy suffices
- Spectra are chimeric (co-eluting), where FIDDLE's performance degrades for reasons
  unrelated to compound polarity
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_2269
tools:
- name: FIDDLE
  role: Provides TCN and rescore checkpoints (Q-TOF and Orbitrap variants) whose top-5
    predictions are stratified by polarity, mass, and superclass
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type whose test-spectra subset is evaluated per LogP polarity stratum
- name: Orbitrap
  role: Instrument type whose test-spectra subset is evaluated per LogP polarity stratum
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:1d721acf8b99c73b55e247af7a875c6bf2bc15e09e0d10c0f4ef75c97dfb61e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Orbitrap
  - Q-TOF
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# logp-polarity-grouping-of-compounds

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies formula-identification accuracy by molecular polarity (canonical SMILES-derived LogP bins), precursor mass bins, and ClassyFire superclass to expose where FIDDLE's top-5 accuracy degrades on NIST23 test spectra.

## When to use

Use when evaluating formula-prediction output on a held-out test set and you need per-compound-property breakdowns (LogP polarity, mass range, chemical superclass) of top-k accuracy rather than a single pooled score.

## When NOT to use

- Spectra come from sources outside the FIDDLE test-compound space (e.g., molecules with atoms beyond C, H, O, N, F, S, Cl, P, B, I, Br, Na, and K, or m/z above 1500), where the annotation join and accuracy comparison are out of scope
- The goal is overall benchmark comparison against SIRIUS or BUDDY rather than property-stratified error analysis — pooled top-1/top-5 accuracy suffices
- Spectra are chimeric (co-eluting), where FIDDLE's performance degrades for reasons unrelated to compound polarity

## Inputs

- Per-spectrum FIDDLE top-5 formula prediction CSV from the NIST23 test-set evaluation (experiments_test_nist23.sh)
- Formula ground-truth labels for test spectra
- Canonical SMILES for test compounds (for LogP and ClassyFire superclass annotation)
- Instrument-type labels (Q-TOF vs. Orbitrap) per spectrum

## Outputs

- Stratified table of top-5 accuracy per LogP polarity group, mass bin, and ClassyFire superclass, per instrument type
- Fig. 3c/d/f-style summary panels showing the inverse LogP trend, mass-bin profile, and superclass comparison
- Flagged list of underperforming superclasses (organohalogens, polymers, organosulfur, unknown)

## How to apply

Run the released FIDDLE TCN and rescore checkpoints (Orbitrap and Q-TOF variants) over held-out NIST23 test spectra via the repository's experiments_test_nist23.sh evaluation script, writing per-spectrum top-5 predictions to a results CSV. Join each spectrum's formula ground truth with per-compound annotations derived from canonical SMILES: LogP (binned into polar/hydrophilic vs. nonpolar/lipophilic polarity groups), precursor mass bins (e.g., <200 Da vs. larger), and ClassyFire superclass. Compute top-5 accuracy per stratum separately for Q-TOF and Orbitrap instrument types and generate Fig. 3c/d/f-style summary panels. Rationale: accuracy is not uniform — polar compounds reach 95.0% top-5 accuracy vs. 88.8% for nonpolar compounds (a 6.2% gap consistent across both instruments) — so pooled metrics mask property-dependent failure modes such as organohalogens, polymers, organosulfur, and unknown superclasses.

## Related tools

- **FIDDLE** (Provides TCN and rescore checkpoints (Q-TOF and Orbitrap variants) whose top-5 predictions are stratified by polarity, mass, and superclass) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type whose test-spectra subset is evaluated per LogP polarity stratum)
- **Orbitrap** (Instrument type whose test-spectra subset is evaluated per LogP polarity stratum)

## Examples

```
bash experiments_test_nist23.sh
```

## Evaluation signals

- Polar-compound top-5 accuracy ≈ 95.0%, moderate ≈ 94.7%, nonpolar ≈ 88.8%, with the ~6.2% gap reproduced for both Q-TOF and Orbitrap
- Stratified table reproduces reported <200 Da top-5 accuracy values of 97.3% (Q-TOF) and 95.0% (Orbitrap)
- Underperforming superclass list matches the paper's flags: organohalogens, polymers, organosulfur, and unknown superclasses
- Every test spectrum joins to exactly one LogP bin, mass bin, and superclass; no spectra dropped silently in the join
- Pooled top-5 accuracy over all strata (93.6% on the evaluation set) is recovered when averaging the stratified table

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data, so strata with sparse coverage may be noisy
- Stratification depends on external LogP and ClassyFire annotations; compounds lacking canonical SMILES annotations cannot be binned
- Performance degrades on chimeric (co-eluting) spectra as spectral mixing increases, which can confound property-based comparisons if mixed spectra enter a stratum

## Evidence

- [results] Inverse LogP accuracy trend: [evidence span withheld — anchor status: not_found]
