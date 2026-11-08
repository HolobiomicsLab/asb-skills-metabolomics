---
name: mgf-spectra-noise-filtering-and-preprocessing
description: Use when raw MGF tandem MS/MS spectra contain low-abundance noise peaks
  or crowded m/z regions and need deterministic cleaning (intensity and m/z cutoffs,
  3-Da de-clustering) before pair-based spectral similarity modeling.
when_to_use_negative:
- The input is already an h5py-format spectrum database produced by ms_convert_mgf2h5py.py
  — re-filtering would change peak lists and break element-wise reproducibility of
  downstream matrices
- Spectra come from an acquisition/proteomics pipeline with its own peak picking and
  de-isotoping thresholds that conflict with the 0.1% intensity / m/z 10–1000 Da /
  3-Da window rules
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyteomics
  role: Parses MGF spectrum files to extract peaks and precursor m/z for filtering
  repo: github.com/banhdzui/TransExION
- name: TransExION
  role: Defines the filtering rules and consumes the cleaned spectra via its conversion
    and training scripts
  repo: github.com/banhdzui/TransExION
- name: h5py
  role: Storage format for the converted, filtered spectrum databases used at large
    scale
  repo: github.com/banhdzui/TransExION
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:f9420be94bf1c2edccb978ec156571465408bbcb2a36896f26d41fd3d8b8ce02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mgf-spectra-noise-filtering-and-preprocessing

## Summary

Noise-filter and normalize MS/MS spectra from MGF files (e.g. GNPS_MassBank_train.mgf from Zenodo record 8175528) into the cleaned, rounded peak lists that TransExION uses to build its mass-difference and neutral-loss input representations.

## When to use

Use when raw MGF tandem MS/MS spectra contain low-abundance noise peaks or crowded m/z regions and need deterministic cleaning (intensity and m/z cutoffs, 3-Da de-clustering) before pair-based spectral similarity modeling.

## When NOT to use

- The input is already an h5py-format spectrum database produced by ms_convert_mgf2h5py.py — re-filtering would change peak lists and break element-wise reproducibility of downstream matrices
- Spectra come from an acquisition/proteomics pipeline with its own peak picking and de-isotoping thresholds that conflict with the 0.1% intensity / m/z 10–1000 Da / 3-Da window rules

## Inputs

- MGF-format MS/MS spectrum files (e.g. GNPS_MassBank_train.mgf, GNPS_MassBank_val.mgf from Zenodo record 8175528)
- Per-spectrum precursor m/z values parsed from MGF headers

## Outputs

- Cleaned peak lists with rounded m/z and relative-abundance intensities (noise peaks removed)
- h5py-format spectrum database files (e.g. GNPS_MassBank_train.db) produced by ms_convert_mgf2h5py.py

## How to apply

Parse each MGF spectrum with pyteomics/mgf, converting peaks to rounded m/z values and relative-abundance intensities. Apply the noise filters described in the TransExION workflow: drop fragment peaks below 0.1% relative intensity and outside the m/z window 10–1000 Da; within a 3-Da sliding window keep only the highest peak. Convert the filtered spectra to h5py format using ms_convert_mgf2h5py.py so that downstream TransExION scripts (training, neutral-loss MDM construction) can consume them efficiently at large scale. The rationale is that the transformer similarity model expects sparse, noise-free peak lists, and identical filtering guarantees recomputed matrices match element-wise during validation.

## Related tools

- **pyteomics** (Parses MGF spectrum files to extract peaks and precursor m/z for filtering) — github.com/banhdzui/TransExION
- **TransExION** (Defines the filtering rules and consumes the cleaned spectra via its conversion and training scripts) — github.com/banhdzui/TransExION
- **h5py** (Storage format for the converted, filtered spectrum databases used at large scale) — github.com/banhdzui/TransExION

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db
```

## Evaluation signals

- No retained peak falls below 0.1% relative intensity or outside the 10–1000 Da m/z range
- No two retained peaks lie within the 3-Da sliding window — the highest peak within each window is kept
- Round-tripping: re-running the filtering on the same MGF yields element-wise identical peak lists (within two-decimal rounding tolerance)
- Output h5py file loads with h5py and contains one entry per MGF spectrum with matching peak counts

## Limitations

- The 3-Da sliding window is a nominal-mass de-clustering heuristic; it may merge closely spaced high-resolution peaks and lose fine isotope structure
- Dropping fragments below 0.1% relative intensity can remove weak but structurally informative peaks for sparse spectra
- Filtering parameters are specific to the GNPS/MassBank training data preparation described for TransExION

## Evidence

- [other] drop fragments <0.1% intensity or m/z <10 or >1000 Da, keep only the highest peak within a 3-Da sliding window: [evidence span withheld — anchor status: not_found]
- [readme] Our library supports a function to convert a MGF format file into h5py format. To do the conversion, run the script ms_convert_mgf2h5py.py with parameters.: "Our library supports a function to convert a MGF format file into h5py format. To do the conversion, run the script"
- [readme] The data we used to train the model can be found in [zenodo link](https://zenodo.org/record/8175528).: "The data we used to train the model can be found in [zenodo link](https://zenodo.org/record/8175528)"
- [other] keeping rounded m/z and relative-abundance intensities after the described noise filtering: [evidence span withheld — anchor status: not_found]
