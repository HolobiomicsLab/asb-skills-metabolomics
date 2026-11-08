---
name: ms2-spectrum-preprocessing
description: Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned and
  normalized before spectrum clustering or similarity computation, e.g. prior to falcon
  DBSCAN clustering of proteomics spectra.
when_to_use_negative:
- 'Clustering metabolomics or top-down data with the bottom-up defaults: the README
  states default min_peaks/min_mz_range settings should be reduced (e.g. lower than
  5 peaks and 250 m/z range) for metabolomics data'
- Spectra are already preprocessed (precursor peaks, noise peaks, and intensity scaling
  already applied) — reapplying defaults could over-filter
- The task is raw-file conversion rather than spectrum cleaning — Thermo raw files
  must first be converted to MGF/mzML (e.g. with ThermoRawFileParser 1.2.3) before
  these filters apply
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3522
tools:
- name: falcon
  role: spectrum clustering tool whose preprocessing module applies precursor/noise
    peak removal, min_peaks/min_mz_range filtering, and intensity scaling before clustering
  repo: https://github.com/bittremieux/falcon
- name: spectrum_utils
  role: spectrum processing library installed alongside falcon (pinned version 0.3.5)
    for MS/MS spectrum manipulation
- name: ThermoRawFileParser 1.2.3
  role: converts Thermo raw files (e.g. PRIDE PXD000561) to MGF peak files that are
    the upstream input to preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:148aad939af533210f323bb7e653c5f512eb7d5b1f98a0b59fe29da1769b9706
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - Pyteomics
  - Python 3.8
  - Scikit-Learn
  - ThermoRawFileParser
  - ThermoRawFileParser 1.2.3
  - falcon
  - fastcluster
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# ms2-spectrum-preprocessing

## Summary

Preprocess MS/MS spectra prior to spectrum clustering by removing precursor-ion and low-intensity noise peaks, discarding low-quality spectra, and transforming peak intensities. As described in the falcon paper (Bittremieux et al., 2021, Rapid Commun. Mass Spectrom.), this preprocessing directly affects downstream cosine-distance clustering purity.

## When to use

Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned and normalized before spectrum clustering or similarity computation, e.g. prior to falcon DBSCAN clustering of proteomics spectra.

## When NOT to use

- Clustering metabolomics or top-down data with the bottom-up defaults: the README states default min_peaks/min_mz_range settings should be reduced (e.g. lower than 5 peaks and 250 m/z range) for metabolomics data
- Spectra are already preprocessed (precursor peaks, noise peaks, and intensity scaling already applied) — reapplying defaults could over-filter
- The task is raw-file conversion rather than spectrum cleaning — Thermo raw files must first be converted to MGF/mzML (e.g. with ThermoRawFileParser 1.2.3) before these filters apply

## Inputs

- MS/MS peak files in mzML, mzXML, or MGF format (e.g. converted from Thermo raw files with ThermoRawFileParser)
- falcon preprocessing settings: min_peaks, min_mz_range, min_mz, max_mz, scaling

## Outputs

- Cleaned MS/MS spectra with precursor-ion and noise peaks removed and intensities transformed, ready for 0.05 m/z binning and feature hashing
- Discarded low-quality spectra (fewer than min_peaks peaks, or min_mz_range m/z span)

## How to apply

Remove peaks corresponding to the precursor ion and low-intensity noise peaks (noise peaks below 10% of the base peak intensity in the falcon benchmark workflow), and restrict spectra to their 50 most intense peaks. Discard low-quality spectra that have fewer than five peaks or a mass range between their minimum and maximum peak m/z less than 250 Da after peak removal. Transform peak intensities with a square-root transformation to de-emphasize overly dominant peaks; note that the scaling method (square root, logarithm, rank, or none) influences the cosine distance threshold eps, so scaling and eps must be chosen together. In falcon these filters are exposed as min_peaks, min_mz_range, min_mz, max_mz, and scaling parameters, with defaults (min 5 peaks, 250 m/z range, min_mz 101, max_mz 500) intended for bottom-up proteomics. Preprocessed spectra are then binned at 0.05 m/z and feature-hashed with MurmurHash3 into low-dimensional vectors for nearest neighbor indexing.

## Related tools

- **falcon** (spectrum clustering tool whose preprocessing module applies precursor/noise peak removal, min_peaks/min_mz_range filtering, and intensity scaling before clustering) — https://github.com/bittremieux/falcon
- **spectrum_utils** (spectrum processing library installed alongside falcon (pinned version 0.3.5) for MS/MS spectrum manipulation)
- **ThermoRawFileParser 1.2.3** (converts Thermo raw files (e.g. PRIDE PXD000561) to MGF peak files that are the upstream input to preprocessing)

## Examples

```
pip install falcon-ms spectrum-utils==0.3.5; falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- falcon.csv contains a cluster label line for each input MS/MS spectrum (discarded low-quality spectra excluded per min_peaks/min_mz_range filters)
- At cosine distance threshold eps 0.05, the fraction of incorrectly clustered spectra is approximately 1% as reported for the human draft proteome dataset
- No spectrum with fewer than 5 peaks or an m/z range below 250 Da survives into the clustering stage
- Precursor-ion peaks and peaks outside min_mz 101 / max_mz 500 m/z are absent from preprocessed spectra
- Changing the scaling method (none vs square root) is accompanied by a corresponding re-check of the eps threshold, since scaling influences eps

## Limitations

- Default preprocessing settings are intended for bottom-up proteomics data; metabolomics and top-down data require reduced min_peaks/min_mz_range and likely adjusted other settings
- The influence of MS/MS spectrum preprocessing on various similarity measures was not evaluated in the study (open question noted in the discussion)
- The scaling method influences the cosine threshold eps, so a fixed eps cannot be reused unchanged across different scaling choices

## Evidence
