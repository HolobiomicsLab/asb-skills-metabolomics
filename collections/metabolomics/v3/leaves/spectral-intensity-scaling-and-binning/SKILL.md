---
name: spectral-intensity-scaling-and-binning
description: Use when raw MS/MS spectra (m/z–intensity peak lists from MoNA/NIST libraries
  or in-house data) must be converted into fixed-length feature vectors for training
  or inference with a neural-network fingerprint predictor such as MetFID.
when_to_use_negative:
- When spectra are already in a fixed-length feature representation (e.g. binned intensity
  vectors) — applying scaling/filtering again would corrupt the data.
- When the downstream task is clustering or library comparison rather than neural-network
  fingerprint prediction, and raw peak lists are preferred for interpretability.
- When spectra have fewer than five peaks with relative intensity above 2% — they
  will be removed anyway and no useful vector can be produced.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenBabel
  role: Used upstream to determine molecular fingerprints (MACCS, FP3, FP4) that pair
    with the binned spectral features for ANN training
- name: MoNA
  role: Source of the MS/MS spectra (peak lists with instrument and collision-energy
    metadata) that are scaled, filtered, and binned
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Source of the MS/MS spectra used alongside MoNA for training-set construction
    and the PCA clustering validation of binning-based features
- name: InChIKey
  role: Compound identifier used to merge spectra across collision energies before
    scaling and binning
- name: MetFID
  role: Downstream CNN/ANN model that consumes the scaled-and-binned spectral feature
    vectors as input
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:0c1a262d0e9f7a285bf31191c6b047f435c0d75dc492e4260ad979d276f8b0c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  - OpenBabel
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spectral-intensity-scaling-and-binning

## Summary

Pre-processing of MS/MS spectra for ML-based fingerprint prediction: scale peak intensities to a 0–100 range, filter out spectra with fewer than five peaks above 2% relative intensity, denoise by removing peaks below 10 intensity, then bin the m/z range into prespecified bins, using 0.1 m/z merging and loss-feature calculation as input steps.

## When to use

Use when raw MS/MS spectra (m/z–intensity peak lists from MoNA/NIST libraries or in-house data) must be converted into fixed-length feature vectors for training or inference with a neural-network fingerprint predictor such as MetFID.

## When NOT to use

- When spectra are already in a fixed-length feature representation (e.g. binned intensity vectors) — applying scaling/filtering again would corrupt the data.
- When the downstream task is clustering or library comparison rather than neural-network fingerprint prediction, and raw peak lists are preferred for interpretability.
- When spectra have fewer than five peaks with relative intensity above 2% — they will be removed anyway and no useful vector can be produced.

## Inputs

- Merged MS/MS spectra peak lists (m/z–intensity pairs) keyed by InChIKey, with precursor mass and ionization mode (positive/negative)
- Spectral-library entries from MoNA libraries (e.g. Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) and NIST 17
- Precursor-ion m/z values for calculating loss features

## Outputs

- Fixed-length binned intensity feature vectors (spectral features) suitable as ANN input
- Filtered training set in which every retained spectrum has at least five peaks with relative intensity above 2%
- Reduced bin feature matrix with all-zero bins removed across the training set

## How to apply

Start from merged per-compound spectra (InChIKey-keyed, with peaks merged across collision energies when their m/z difference is 0.1 or less) and calculate loss features as the m/z difference between the precursor ion and each peak. Scale all peak intensities so values lie between 0 and 100, remove peaks whose intensity is lower than 10, and drop entire spectra that have fewer than five peaks with relative intensity above 2%. Next, bin the m/z range of each spectrum into prespecified bins (continuous integer m/z bins in the PCA clustering analysis), accumulating intensities within each bin as feature values, and discard bins consisting of all 0's across all spectra in the training set. The rationale is to standardize heterogeneous library spectra (different instruments such as HCD, QqQ, Q-TOF and IT, and different collision energies) into a comparable, fixed-length input for the ANN; removing all-zero bins reduces input dimensionality without losing signal present in the training data.

## Related tools

- **OpenBabel** (Used upstream to determine molecular fingerprints (MACCS, FP3, FP4) that pair with the binned spectral features for ANN training)
- **MoNA** (Source of the MS/MS spectra (peak lists with instrument and collision-energy metadata) that are scaled, filtered, and binned) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Source of the MS/MS spectra used alongside MoNA for training-set construction and the PCA clustering validation of binning-based features)
- **InChIKey** (Compound identifier used to merge spectra across collision energies before scaling and binning)
- **MetFID** (Downstream CNN/ANN model that consumes the scaled-and-binned spectral feature vectors as input) — https://github.com/aohongyu/MetFID

## Examples

```
python3 main.py
```

## Evaluation signals

- Every retained spectrum in the output feature matrix has at least five peaks with relative intensity above 2%; spectra failing this are absent.
- All peak intensity values in the scaled spectra fall within the [0, 100] range.
- No bin in the final training feature matrix consists of all 0's across all spectra in the training set.
- The binned feature dimensionality equals the number of non-empty bins after removal, not the raw m/z range.
- In the PCA clustering analysis, spectra generated using similar collision energies tend to cluster together, and HCD/QqQ collision-cell spectra co-cluster with IT spectra.

## Limitations

- Spectra with fewer than five peaks above 2% relative intensity are discarded entirely, so very sparse library spectra cannot be used.
- Binning is performed with prespecified bins; the PCA analysis found 30 eV to be a reasonable cutoff for high and low collision energies, so collision-energy heterogeneity must be accounted for separately (separate ANNs for low vs high energy).
- Peaks are merged across spectra only when their m/z difference is 0.1 or less; larger differences are not merged and may fragment identical peaks across bins.
- Scaling and filtering operate on merged spectra keyed by InChIKey; compounds without a resolvable InChIKey cannot be merged.

## Evidence
