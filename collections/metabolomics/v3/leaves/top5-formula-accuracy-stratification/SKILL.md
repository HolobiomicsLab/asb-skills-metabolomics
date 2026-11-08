---
name: top5-formula-accuracy-stratification
description: Use when you need to characterize how a formula-prediction model's top-5
  accuracy varies with compound polarity (LogP), precursor mass (e.g., <200 Da bins),
  or ClassyFire superclass on tandem MS/MS test spectra.
when_to_use_negative:
- You need overall top-1/top-5 benchmark accuracy against SIRIUS or BUDDY — use the
  standard NIST23 evaluation instead of property-stratified analysis.
- Input spectra are chimeric (co-eluting) or come from external metabolomics datasets
  with no SMILES/superclass annotations available for stratification.
- Compounds contain atoms outside the frequent-element set (C, H, O, N, F, S, Cl,
  P, B, I, Br, Na, K) or exceed the 300-atom cap, since FIDDLE's test space excludes
  them.
tools:
- name: FIDDLE
  role: Predicts candidate molecular formulas from MS/MS spectra via the TCN and rescore
    checkpoints; the repository's experiments_test_nist23.sh script produces per-spectrum
    prediction CSVs for stratified scoring
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package exposing pre-trained FIDDLE models for batched prediction from
    MGF files or Python arrays when rerunning inference over the test set
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:84af85c63d6f63b3eebca2e15c8b27c37eb81b795a7d0f9d7ddc3cdfd1a50cc6
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
  - Q-TOF
  - msfiddle
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# top5-formula-accuracy-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies FIDDLE's top-5 molecular formula identification accuracy over the NIST23 held-out test set by molecular properties — LogP-derived polarity, precursor mass bins, and ClassyFire superclass — for both Q-TOF and Orbitrap instrument types.

## When to use

Use when you need to characterize how a formula-prediction model's top-5 accuracy varies with compound polarity (LogP), precursor mass (e.g., <200 Da bins), or ClassyFire superclass on tandem MS/MS test spectra.

## When NOT to use

- You need overall top-1/top-5 benchmark accuracy against SIRIUS or BUDDY — use the standard NIST23 evaluation instead of property-stratified analysis.
- Input spectra are chimeric (co-eluting) or come from external metabolomics datasets with no SMILES/superclass annotations available for stratification.
- Compounds contain atoms outside the frequent-element set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or exceed the 300-atom cap, since FIDDLE's test space excludes them.

## Inputs

- Held-out NIST23 test MS/MS spectra (Q-TOF and Orbitrap variants)
- Pre-trained FIDDLE TCN and rescore checkpoints (v1.0.0 release weights)
- Per-spectrum prediction CSV (predicted candidate formulas with confidence scores)
- Ground-truth molecular formulas plus canonical SMILES with derived LogP values
- ClassyFire superclass annotations for test compounds

## Outputs

- Per-stratum top-5 accuracy table split by polarity group, mass bin, superclass, and instrument type
- Fig. 3c/d/f-style summary panels (LogP trend, mass-bin profile, superclass comparison)
- Flagged underperforming superclass list (e.g., organohalogens, polymers, organosulfur, unknown)

## How to apply

Run the released FIDDLE TCN and rescore checkpoints (Orbitrap and Q-TOF variants) over the held-out NIST23 test spectra using the repository's evaluation script (experiments_test_nist23.sh), writing per-spectrum predictions to a results CSV. Parse the CSV and join each spectrum's ground-truth formula with molecular annotations derived from canonical SMILES: LogP (binned into hydrophilic/polar, moderate, and lipophilic/nonpolar groups) and ClassyFire superclass. Also bin spectra by precursor mass (e.g., <200 Da vs. larger bins) and compute top-5 accuracy per stratum, separately for Q-TOF (10 ppm precursor tolerance) and Orbitrap (5 ppm tolerance) data. Generate Fig. 3c/d/f-style panels summarizing the inverse LogP trend, the mass-bin profile, and the superclass comparison. Validate the stratified table against the reported anchor values: 97.3% Q-TOF and 95.0% Orbitrap top-5 accuracy below 200 Da, and the paper's underperforming superclass list (organohalogens, polymers, organosulfur, unknown).

## Related tools

- **FIDDLE** (Predicts candidate molecular formulas from MS/MS spectra via the TCN and rescore checkpoints; the repository's experiments_test_nist23.sh script produces per-spectrum prediction CSVs for stratified scoring) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package exposing pre-trained FIDDLE models for batched prediction from MGF files or Python arrays when rerunning inference over the test set) — https://github.com/josiehong/msfiddle

## Examples

```
conda env create -f environment.yml && bash experiments_test_nist23.sh  # run FIDDLE checkpoints over NIST23 test spectra to produce the per-spectrum prediction CSV for stratified top-5 scoring
```

## Evaluation signals

- Polar (highest-LogP-inverse) group reproduces 95.0% top-5 accuracy with a 6.2% gap to nonpolar (88.8%), consistent across Q-TOF and Orbitrap
- Sub-200 Da mass bin reproduces the reported 97.3% (Q-TOF) and 95.0% (Orbitrap) top-5 accuracy values
- Underperforming superclasses match the paper's list: organohalogens, polymers, organosulfur, and unknown
- Test-set join yields no compounds shared with the training set (canonical-SMILES-based split invariant)
- Each spectrum carries a single instrument type so strata can be reported separately for 10 ppm (Q-TOF) and 5 ppm (Orbitrap) tolerance settings

## Limitations

- The test split is by canonical SMILES, so stratified accuracy may reflect compounds absent from training rather than intrinsic property difficulty
- Confidence-score FDR thresholds are proposed but not implemented, so per-stratum ranking is not error-rate controlled
- Findings apply to singly-dominated, common precursor types in the (0, 1500] m/z range; adduct type is not predicted and chimeric spectra degrade performance

## Evidence

- [results] Top-5 accuracy inversely correlates with LogP: [evidence span withheld — anchor status: not_found]
