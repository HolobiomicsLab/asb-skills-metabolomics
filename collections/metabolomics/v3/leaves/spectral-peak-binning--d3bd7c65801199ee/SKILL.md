---
name: spectral-peak-binning
description: Use when raw merged MS/MS peak lists (m/z–intensity pairs) must be converted
  into a uniform binned feature matrix for machine-learning models such as the MetFID
  fingerprint predictor.
when_to_use_negative:
- The spectra are still raw/unmerged per collision energy — MetFID merges spectra
  of the same compound at different collision energies first, because cluster analysis
  showed spectra generated using similar collision energies cluster together and 30
  eV was a reasonable cutoff for high/low collision energies.
- The input is already a fixed-length binned feature table or a spectral_feature_table.csv
  — re-binning would discard the all-zero bin filtering already applied.
- Only a handful of spectra need simple library matching — spectral matching does
  not require constructing an ANN training feature table.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetFID
  role: Produces the binned m/z feature table and trains the CNN/ANN fingerprint predictor
    on it
  repo: https://github.com/aohongyu/MetFID
- name: PyFingerprint
  role: Supplies fingerprint types (MACCS, FP3, FP4 via OpenBabel) whose prediction
    depends on the binned spectrum features
  repo: https://github.com/hcji/PyFingerprint
- name: Open Babel
  role: Determines MACCS, FP3, and FP4 fingerprints for the 528-entry label vector
    that the binned features are trained to predict
  repo: https://openbabel.org/wiki/Python
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:95ba8d304872382f69cc17276971d7231cdc20b24180e15f649be8c2af8947e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MoNA
  - NIST 17
  - Open Babel
  - PyFingerprint
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spectral-peak-binning

## Summary

Discretizes an MS/MS spectrum's m/z range into prespecified continuous integer bins with accumulated peak intensities as feature values, producing a fixed-length numeric feature table (spectral_feature_table.csv) used to train MetFID's ANN/CNN fingerprint predictor; all-zero bins across the training set are dropped.

## When to use

Use when raw merged MS/MS peak lists (m/z–intensity pairs) must be converted into a uniform binned feature matrix for machine-learning models such as the MetFID fingerprint predictor.

## When NOT to use

- The spectra are still raw/unmerged per collision energy — MetFID merges spectra of the same compound at different collision energies first, because cluster analysis showed spectra generated using similar collision energies cluster together and 30 eV was a reasonable cutoff for high/low collision energies.
- The input is already a fixed-length binned feature table or a spectral_feature_table.csv — re-binning would discard the all-zero bin filtering already applied.
- Only a handful of spectra need simple library matching — spectral matching does not require constructing an ANN training feature table.

## Inputs

- Denoised, intensity-scaled, and merged MS/MS peak lists per compound (m/z, intensity pairs) keyed by InChIKey
- Precursor ion m/z and ionization mode (positive/negative) metadata for each merged spectrum
- Training-set collection of spectra from MoNA libraries (Vaniya/Fiehn, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) and NIST 17

## Outputs

- Binned feature matrix in which each row is one spectrum and each column is a continuous integer m/z bin with accumulated intensity values (spectral_feature_table.csv)
- Set of retained bin columns after dropping all-zero bins across the training set

## How to apply

After intensity scaling to 0–100, denoising, and merging spectra of the same compound (InChIKey; peaks within 0.1 m/z combined, mean intensity assigned), compute loss features as the difference between precursor ion m/z and each peak's m/z. Then bin the m/z range of each MS/MS spectrum into prespecified bins: continuous integer m/z bins whose feature values are the accumulated (summed) intensities falling in each bin. The precursor mass and the loss features are appended so the ANN sees precursor mass, ionization mode, loss features, and binned spectrum. Finally, remove bins that consist of all 0's across all spectra in the training set — the retained bin columns are written to spectral_feature_table.csv for training. This binning gives every spectrum the same vector length, which is required because MetFID's multi-class multi-label ANN/CNN predicts the full 528-entry fingerprint (MACCS + FP3 + FP4) in one pass.

## Related tools

- **MetFID** (Produces the binned m/z feature table and trains the CNN/ANN fingerprint predictor on it) — https://github.com/aohongyu/MetFID
- **PyFingerprint** (Supplies fingerprint types (MACCS, FP3, FP4 via OpenBabel) whose prediction depends on the binned spectrum features) — https://github.com/hcji/PyFingerprint
- **Open Babel** (Determines MACCS, FP3, and FP4 fingerprints for the 528-entry label vector that the binned features are trained to predict) — https://openbabel.org/wiki/Python

## Examples

```
python3 main.py
```

## Evaluation signals

- Every retained column in spectral_feature_table.csv has at least one non-zero value across the training set; no all-zero bins remain (per the paper's rule 'We removed bins that consisted of all 0's across all the spectra in the training set').
- All spectra map to the same vector length after binning; the number of columns equals the number of non-empty integer m/z bins.
- Intensity scaling precedes binning so intensities are bounded 0–100; spectra with fewer than five peaks above 2% relative intensity were removed before binning.
- Merged-peak m/z differences are ≤ 0.1 within each InChIKey group — bin merging happens after this merge, not before.
- Loss features equal precursor m/z minus peak m/z for each peak, using peak intensity as the proxy.

## Limitations

- Binning is defined on the training set; all-zero bins are dropped, so a new test spectrum with peaks only in dropped bins loses that information (bin set is not re-derived per prediction).
- The skill assumes peaks within 0.1 m/z have already been merged; bin width interacts with this 0.1 m/z merge threshold.
- Prior work (MetFrag, ChemDistiller, CSI:FingerID comparison) and the authors note that the target-decoy approach can help estimate false discovery rates in downstream ranking — binning itself does not address specificity of candidate ranking.

## Evidence
