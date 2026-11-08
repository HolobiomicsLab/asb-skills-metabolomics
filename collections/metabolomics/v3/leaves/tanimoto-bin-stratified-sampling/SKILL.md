---
name: tanimoto-bin-stratified-sampling
description: Use when preparing training pairs for MS2DeepScore 2.0 — before fitting
  the Siamese network, to prevent over-sampling of common molecules or easy Tanimoto
  ranges and enforce equal pair counts per Tanimoto bin.
when_to_use_negative:
- When you have fewer than ~100,000 spectra without checking/optimizing pair sampling
  — the README warns 'Particularly for smaller training sets, the pair sampling can
  be suboptimal if not checked'
- When the goal requires balanced ionization-mode pair sampling — the article states
  'our new sampling algorithm did not enforce balanced sampling of the different ionization
  mode pairs'
- When spectra lack reliable structural annotation (InChIKeys), since Tanimoto bin
  assignment depends on it
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore 2.0
  role: The new pair sampling algorithm is implemented inside MS2DeepScore 2.0's training
    pipeline to reduce biases during model training
  repo: github.com/matchms/MS2DeepScore
- name: PyTorch
  role: Backend framework in which the entire MS2DeepScore model (and by extension
    its training pipeline) is implemented
- name: matchms
  role: Recommended for cleaning spectra before training; provides the DEFAULT_FILTERS
    pipeline used alongside MS2DeepScore
  repo: github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:78f508766e3a6a13c6f77c68ca410a5667605d9930ec630755eac25523631f33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tanimoto-bin-stratified-sampling

## Summary

A training-pair sampling strategy for MS2DeepScore 2.0 that draws spectrum pairs equally across the Tanimoto score range grouped into 10 bins while capping per-molecule sampling frequency disparity below 15%, reducing biases introduced during Siamese network training.

## When to use

Use when preparing training pairs for MS2DeepScore 2.0 — before fitting the Siamese network, to prevent over-sampling of common molecules or easy Tanimoto ranges and enforce equal pair counts per Tanimoto bin.

## When NOT to use

- When you have fewer than ~100,000 spectra without checking/optimizing pair sampling — the README warns 'Particularly for smaller training sets, the pair sampling can be suboptimal if not checked'
- When the goal requires balanced ionization-mode pair sampling — the article states 'our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs'
- When spectra lack reliable structural annotation (InChIKeys), since Tanimoto bin assignment depends on it

## Inputs

- MGF or other spectrum library file (msp, mzml, mgf, mzxml, json, usi supported by matchms) with cleaned spectra annotated with InChIKeys and ionization modes
- Structural fingerprints / InChIKeys for computing Tanimoto scores between candidate molecule pairs
- Training settings from SettingsMS2Deepscore (e.g. ionisation_mode='both')

## Outputs

- Training pair set with pairs sampled exactly equally across the 10 Tanimoto score bins
- Per-molecule sampling counts constrained to <15% disparity from the maximum sampling count
- Sampling report (per-bin pair counts, per-molecule counts, max-vs-average count ratio) to verify balance

## How to apply

Load the annotated spectra with InChIKeys and ionization modes, compute Tanimoto scores between candidate pairs from their structural fingerprints, and partition the score range into 10 bins. Run the pair sampler that tracks a sampling count for every molecule and rejects candidate pairs whose inclusion would push a molecule's count above the maximum sampling count, keeping per-molecule frequency disparity under 15% of the average. Iteratively draw pairs per bin until each bin target is met or the per-molecule cap is hit, recording which molecules and ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) were selected per bin. Write a sampling report with per-bin pair counts, per-molecule sampling counts, and the max-vs-average count ratio. Inspect this report before training: the README notes pair sampling quality 'has to be checked and potentially re-optimized for new datasets', especially for smaller training sets. Note the algorithm balances Tanimoto bins and molecule frequency but does NOT enforce balanced sampling across ionization-mode pairs — if negative vs. negative mode performance matters, this is an acknowledged gap.

## Related tools

- **MS2DeepScore 2.0** (The new pair sampling algorithm is implemented inside MS2DeepScore 2.0's training pipeline to reduce biases during model training) — github.com/matchms/MS2DeepScore
- **PyTorch** (Backend framework in which the entire MS2DeepScore model (and by extension its training pipeline) is implemented)
- **matchms** (Recommended for cleaning spectra before training; provides the DEFAULT_FILTERS pipeline used alongside MS2DeepScore) — github.com/matchms/matchms

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', ionisation_mode='both')
```

## Evaluation signals

- Verify each of the 10 Tanimoto bins contains training pairs with equal counts (the article states pairs are 'sampled exactly equally across the Tanimoto score range grouped into 10 bins')
- Check that every molecule's sampling count stays within <15% disparity relative to the maximum sampling count
- Confirm the sampling report includes per-bin pair counts and max-vs-average count ratio statistics
- Cross-check that positive-positive, negative-negative, and positive-negative ionization-mode pair combinations are present in the sampled training set
- Inspect the pair-sampling tutorial output before training — README: 'the quality of the pair sampling has to be checked and potentially re-optimized for new datasets'

## Limitations

- Balanced Tanimoto binning is enforced, but the sampling does not guarantee equal pair counts per Tanimoto bin for each ionization mode combination, since Tanimoto score distributions differ between modes
- Not enforcing balanced sampling over ionization modes 'might improve model performance for negative vs. negative ionization mode predictions' if corrected in future work
- Pair sampling must be checked and potentially re-optimized per dataset, especially for smaller training sets
- Training your own model is only recommended for users with ML familiarity and >100,000 spectra of sufficiently diverse types

## Evidence

- [discussion] Pair sampling algorithm balances pairs over Tanimoto bins and molecule frequency: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [intro] Reducing training bias via pair sampling: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [other] Task card finding on <15% disparity: [evidence span withheld — anchor status: not_found]
- [readme] Pair sampling must be verified per dataset: [evidence span withheld — anchor status: external_doc]
- [readme] Training wrapper invocation with settings: "train_ms2deepscore_wrapper(
    settings, 
    SettingsEmbeddingEvaluator()"
