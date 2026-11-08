---
name: spectral-library-curation
description: Use when preparing a training-ready spectral feature table from multiple
  MS/MS spectra per compound (e.g. MoNA Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank,
  Fiehn HILIC, or NIST 17) and raw spectra differ in collision energy, ion mode, or
  adduct.
when_to_use_negative:
- Input is already a curated, single-spectrum-per-compound feature table — the merge
  and denoise steps would be redundant or harmful.
- Analyzing experimental analyte spectra for candidate ranking (post-training) rather
  than building the training set — use the MetFID prediction workflow instead.
- Spectra are negative mode, H- or Na+ adducts, or outside 100–1010 Da range — these
  fail the homogeneity criteria and must not be included in the training table.
edam_operation: http://edamontology.org/operation_0283
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: MetFID
  role: Implements the spectral curation and merging pipeline; bins spectra and computes
    loss features for fingerprint training.
  repo: https://github.com/aohongyu/MetFID
- name: MoNA
  role: Source of MS/MS spectra from Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank, Fiehn
    HILIC for curation.
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Source of MS/MS spectra for curation.
- name: InChIKey
  role: Compound identifier used to merge spectra of the same compound across collision
    energies.
- name: OpenBabel
  role: Determines MACCS/FP3/FP4 fingerprints (528-bit vector) for compounds after
    spectra are curated.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:3eb0accd03bdd20026c8ec3754381e00d1fe14e8521041b9df2a13e595aa38a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  edam_operation: http://edamontology.org/operation_0283
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-library-curation

## Summary

Curate and merge MS/MS spectra from MoNA and NIST 17 libraries into a unified, denoised, binned spectral feature table ready for training a fingerprint predictor. Rationale: raw library spectra vary in collision energy, instrument type, and adduct, so merging same-compound spectra and removing bins of all zeros produces a consistent feature vector. This is why MetFID merges multiple MS/MS spectra per compound before feature extraction.

## When to use

Use when preparing a training-ready spectral feature table from multiple MS/MS spectra per compound (e.g. MoNA Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC, or NIST 17) and raw spectra differ in collision energy, ion mode, or adduct.

## When NOT to use

- Input is already a curated, single-spectrum-per-compound feature table — the merge and denoise steps would be redundant or harmful.
- Analyzing experimental analyte spectra for candidate ranking (post-training) rather than building the training set — use the MetFID prediction workflow instead.
- Spectra are negative mode, H- or Na+ adducts, or outside 100–1010 Da range — these fail the homogeneity criteria and must not be included in the training table.

## Inputs

- MS/MS spectra files from MoNA repository (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC)
- MS/MS spectra files from NIST 17 library
- InChIKey identifiers as compound identifiers

## Outputs

- Binned spectral feature table (spectral_feature_table.csv) with integer m/z bins and accumulated intensities
- Merged, denoised, filtered spectra keyed by InChIKey

## How to apply

Load downloaded MS/MS spectra from MoNA and NIST 17, scale peak intensities to 0–100 and drop spectra with fewer than five peaks of relative intensity above 2%. Denoise by finding the highest-intensity peak whose mass exceeds the precursor mass, removing peaks less intense than it, then discarding all peaks with intensity below 10. Filter to positive-mode, collision-cell (HCD, QqQ, Q-TOF) or ion-trap (IT) spectra in the 100–1010 Da range with H+ or NH4+ adducts only. Merge spectra of the same compound (keyed by InChIKey) that were acquired at different collision energies, combining peaks with m/z difference ≤ 0.1 and assigning the merged peak the mean intensity. Finally, bin each spectrum's m/z range into integer bins, compute loss features as precursor-to-peak m/z differences, remove bins that are all 0’s across all spectra in the training set, and write the binned feature table (spectral_feature_table.csv).

## Related tools

- **MetFID** (Implements the spectral curation and merging pipeline; bins spectra and computes loss features for fingerprint training.) — https://github.com/aohongyu/MetFID
- **MoNA** (Source of MS/MS spectra from Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC for curation.) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Source of MS/MS spectra for curation.)
- **InChIKey** (Compound identifier used to merge spectra of the same compound across collision energies.)
- **OpenBabel** (Determines MACCS/FP3/FP4 fingerprints (528-bit vector) for compounds after spectra are curated.)

## Examples

```
python3 main.py
```

## Evaluation signals

- Verify the output table exists as a CSV with the binned feature distribution matching the paper's reporting (e.g. MoNA: 11,784 spectra / 5,667 compounds; NIST 17: 122,481 spectra / 10,730 compounds before merging).
- Check that all retained spectra meet homogeneity criteria: positive mode, HCD/QqQ/Q-TOF/IT, 100–1010 Da, H+ or NH4+ adduct.
- Confirm no retained feature column is all 0’s across the training set.
- Verify peaks with m/z difference ≤ 0.1 across same-InChIKey spectra were merged to a single peak with mean intensity.
- Ensure every spectrum has ≥ 5 peaks with relative intensity > 2% and that denoising removed peaks below intensity 10.

## Limitations

- Merging assumes same compound's spectra at different collision energies produce comparable peaks (m/z difference ≤ 0.1); cluster analysis showed 30 eV is a reasonable cutoff separating high- and low-collision-energy behaviour.
- Combining IT and HCD spectra does not change prediction performance, but the merged model's accuracy/F1 can drop when tested only with low-energy spectra compared to separate models.
- Spectra from instruments that do not use collision cell or ion trap are excluded, limiting the curation pipeline to those fragmentation types.

## Evidence
