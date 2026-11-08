---
name: neutral-loss-feature-calculation
description: Use when converting a merged MS/MS peak list (m/z–intensity pairs with
  a known precursor m/z) into a feature representation that captures loss information
  for ANN fingerprint prediction.
when_to_use_negative:
- The input spectra have not yet been scaled, denoised, and merged for the same compound
  — applying loss-feature calculation before those steps would produce inconsistent
  loss values across collision energies
- The spectra lack a precursor ion m/z, so the precursor-to-peak difference cannot
  be computed (e.g., spectra from fragmentation mechanisms outside the article's collision-cell
  or ion-trap scope)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetFID
  role: Pipeline whose preprocessing step computes loss features from MS/MS spectra
    before binning and ANN fingerprint prediction
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source of MS/MS spectra (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN
    PlaSMA, MassBank, Fiehn HILIC) that are preprocessed and loss-feature–augmented
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Additional MS/MS spectral library used to train MetFID after the same loss-feature
    preprocessing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:8f2fdbb243e2f2a595b629cd2db4d5a66d25e3bfd20e0c27d057a35da773dde2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# neutral-loss-feature-calculation

## Summary

Derives neutral-loss features from MS/MS peak lists by computing the m/z difference between the precursor ion and each fragment peak (using peak intensity as a proxy), as done in MetFID's preprocessing of MoNA and NIST 17 spectra before binning.

## When to use

Use when converting a merged MS/MS peak list (m/z–intensity pairs with a known precursor m/z) into a feature representation that captures loss information for ANN fingerprint prediction.

## When NOT to use

- The input spectra have not yet been scaled, denoised, and merged for the same compound — applying loss-feature calculation before those steps would produce inconsistent loss values across collision energies
- The spectra lack a precursor ion m/z, so the precursor-to-peak difference cannot be computed (e.g., spectra from fragmentation mechanisms outside the article's collision-cell or ion-trap scope)

## Inputs

- Merged MS/MS peak list per compound (m/z–intensity pairs), after scaling intensities to 0–100, removing spectra with fewer than five peaks above 2% relative intensity, and denoising
- Precursor ion m/z for each spectrum
- InChIKey identifier linking multiple collision-energy spectra of the same compound

## Outputs

- Neutral-loss values (precursor m/z minus each peak m/z) per peak, paired with the peak intensity
- Augmented feature vector combining loss features with binned m/z–intensity features, ultimately written to spectral_feature_table.csv

## How to apply

For each peak in a spectrum that has been scaled to 0–100 intensity, denoised, and merged across collision energies for the same compound (identified by InChIKey), compute a 'loss' value as the difference between the precursor ion's m/z and the peak's m/z; the article treats the peak intensity as a proxy for the loss feature. These loss values accompany the binned m/z–intensity features that are later written to spectral_feature_table.csv, with all-zero bins across the training set dropped. The rationale is that precursor-to-fragment m/z differences encode neutral losses relevant to fingerprint prediction, augmenting the plain m/z-intensity representation.

## Related tools

- **MetFID** (Pipeline whose preprocessing step computes loss features from MS/MS spectra before binning and ANN fingerprint prediction) — https://github.com/aohongyu/MetFID
- **MoNA** (Source of MS/MS spectra (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) that are preprocessed and loss-feature–augmented) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Additional MS/MS spectral library used to train MetFID after the same loss-feature preprocessing)

## Examples

```
python3 main.py  # runs the MetFID pipeline that computes loss features and bins MS/MS spectra from _files/testing_compound.txt
```

## Evaluation signals

- Every peak's loss value equals the precursor m/z minus that peak's m/z; spot-check a few peaks against the raw spectral_feature_table.csv input
- Loss values are non-negative for peaks whose m/z is less than or equal to the precursor m/z; flag spectra with peaks above the precursor m/z before computing losses
- The loss features are paired with each peak's intensity, so the number of loss features equals the number of retained peaks per merged spectrum
- After binning, dropping bins that are all 0's across the training set leaves a feature table whose columns are non-constant, consistent with the article's preprocessing

## Limitations

- The method treats peak intensity as a proxy for the neutral-loss feature; it does not model actual neutral masses or fragment identities
- Loss-feature calculation is only meaningful for spectra within the article's homogeneity criteria (positive mode, HCD/QqQ/Q-TOF or IT fragmentation, 100–1010 Da mass range, H+ or NH4+ adducts)
- Because loss features are derived from merged spectra, they inherit any artifacts of the m/z-merge rule (peaks within 0.1 m/z are averaged)

## Evidence
