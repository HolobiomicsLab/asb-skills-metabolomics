---
name: tandem-mass-spectra-preprocessing
description: Use when raw MS/MS spectra in MGF format must be converted to h5py and
  normalized/denoised before spectral similarity computation, spectral library search,
  or training/testing TransExION on GNPS/GNPS_MassBank/mergedGNPS data.
when_to_use_negative:
- The input is already an h5py (.db) spectrum file that was preprocessed under the
  same protocol — re-running conversion/filtering would alter peak lists inconsistently
  with prior processing.
- The downstream task is structural (Tanimoto) similarity on molecular fingerprints
  via RDKit, not spectral peak-list processing — ms_compute_structural_similarity.py
  is the relevant entry point instead.
- The goal is computing spectral similarity scores themselves (ms_test_spectral_similarity.py
  or baseline Cosine/Spec2Vec/MS2DeepScore scoring) rather than preparing the spectra
  those tools consume.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TransExION
  role: Provides ms_convert_mgf2h5py.py, the MGF-to-h5py conversion utility whose
    output format defines the preprocessed spectrum representation
  repo: github.com/banhdzui/TransExION
- name: h5py
  role: Storage backend for the converted large-scale spectrum files
- name: pyteomics
  role: Python dependency used by the TransExION library for spectrum parsing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:0fef21d075e98c7edc0e8b551c8ce96a281607bae22fbbf467e551351e33b040
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore
  - Matplotlib
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# tandem-mass-spectra-preprocessing

## Summary

Preprocess MS/MS tandem mass spectra into the normalized, denoised h5py representation required by TransExION-style spectral similarity pipelines: convert MGF files to h5py, round m/z to two decimals, convert intensities to relative abundance (%), filter fragments (0.1% ≤ intensity, 10 ≤ m/z ≤ 1000 Da), and denoise by keeping only the highest peak within a 3-Da sliding window.

## When to use

Use when raw MS/MS spectra in MGF format must be converted to h5py and normalized/denoised before spectral similarity computation, spectral library search, or training/testing TransExION on GNPS/GNPS_MassBank/mergedGNPS data.

## When NOT to use

- The input is already an h5py (.db) spectrum file that was preprocessed under the same protocol — re-running conversion/filtering would alter peak lists inconsistently with prior processing.
- The downstream task is structural (Tanimoto) similarity on molecular fingerprints via RDKit, not spectral peak-list processing — ms_compute_structural_similarity.py is the relevant entry point instead.
- The goal is computing spectral similarity scores themselves (ms_test_spectral_similarity.py or baseline Cosine/Spec2Vec/MS2DeepScore scoring) rather than preparing the spectra those tools consume.

## Inputs

- MGF-format MS/MS peak list files (e.g., GNPS_MassBank_train.mgf, GNPS_MassBank_val.mgf, mergedGNPS reference/query spectra)
- Optionally, already-converted h5py spectrum files (ms_convert_mgf2h5py.py output, .db files)

## Outputs

- h5py-format spectrum files (.db) with rounded m/z (2 decimals), relative-abundance intensities (%), fragments filtered to 0.1%–100% intensity and 10–1000 Da m/z, and 3-Da-window denoising applied
- Log/console output from ms_convert_mgf2h5py.py reporting conversion success

## How to apply

First convert MGF peak lists to the large-scale h5py storage format with ms_convert_mgf2h5py.py, since the TransExION library reads h5py rather than raw MGF. Then apply the paper's preprocessing protocol to every spectrum: round m/z values to two decimal places, convert peak intensities to relative abundance (percent of base peak), retain only fragments with relative intensity ≥ 0.1% and m/z between 10 and 1000 Da, and denoise by keeping only the highest peak within each 3-Da sliding window (this suppresses isotopes/near-duplicate peaks). The rationale is that downstream supervised similarity models (TransExION vs. Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) assume this canonical peak representation, so unnormalized intensity scales or unfiltered noise peaks would corrupt similarity estimates. Decision point: if spectra are already in h5py and preprocessed (e.g., reused from the Zenodo record 8175528 training set), skip re-conversion and reuse the files directly. The same preprocessing must be applied identically to reference (training), query (testing), and validation splits so that similarity scores remain comparable.

## Related tools

- **TransExION** (Provides ms_convert_mgf2h5py.py, the MGF-to-h5py conversion utility whose output format defines the preprocessed spectrum representation) — github.com/banhdzui/TransExION
- **h5py** (Storage backend for the converted large-scale spectrum files)
- **pyteomics** (Python dependency used by the TransExION library for spectrum parsing)

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db
```

## Evaluation signals

- The output .db file opens with h5py and contains spectra whose m/z values are rounded to exactly two decimal places.
- No retained fragment falls below 0.1% relative intensity or outside the 10–1000 Da m/z range.
- Within any 3-Da sliding window, only the single highest-intensity peak is retained (near-duplicate peaks removed).
- Peak intensities are relative abundances in percent, not raw instrument counts.
- The same converted files successfully feed ms_test_spectral_similarity.py / ms_train_spectal_similarity.py without format errors, reproducing the paper's pipeline on Zenodo record 8175528 data.

## Limitations

- The 10–1000 Da m/z window and 0.1% intensity cutoff discard low-mass and very low-abundance fragments, which may be informative for some compound classes.
- The 3-Da denoising window can remove genuine low-intensity peaks adjacent to strong peaks; the protocol is fixed per the paper and not tunable per dataset.
- Preprocessing parameters are tied to TransExION's training data (GNPS/GNPS_MassBank-derived); spectra with very different acquisition characteristics may need protocol revalidation.

## Evidence

- [readme] To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format.: "To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
- [readme] Convert mgf files into h5py format files: python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db: "Convert mgf files into h5py format files:
```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db"
- [methods] Preprocess all spectra per the paper's protocol: round m/z to two decimals, convert intensities to relative abundance (%), keep fragments with 0.1% <= intensity and 10 <= m/z <= 1000 Da, and denoise by retaining only the highest peak within a 3-Da sliding window.: [evidence span withheld — anchor status: not_found]
- [abstract] TransExION detects related fragments between MS/MS spectra through their mass difference: "TransExION detects related fragments between MS/MS spectra through their mass difference"
