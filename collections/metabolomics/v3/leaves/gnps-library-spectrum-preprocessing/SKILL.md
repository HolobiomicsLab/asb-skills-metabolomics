---
name: gnps-library-spectrum-preprocessing
description: Use when preparing a downloaded GNPS MGF library (e.g. ALL_GNPS.mgf)
  as training data or as a cleaned search library for MS2Deepscore/Spec2Vec/MS2Query,
  so that spectra are uniform and noise-free.
when_to_use_negative:
- Input is already a cleaned, intensity-normalized library such as the pretrained
  MS2Query sqlite library downloaded via 'ms2query --library ... --download' — re-cleaning
  is unnecessary and may alter peak lists.
- The task is peak picking or clustering of query MS2 spectra from LC-MS/MS runs;
  MS2Query itself does no peak picking, and tools like MZMine/FBMN should preprocess
  feature-level spectra first.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Performs library cleaning, metadata harmonization, peak filtering, intensity
    normalization, and InChIKey lookup via matchms extras
  repo: github.com/iomega/ms2query
- name: MS2Query
  role: Consumes the cleaned library; provides pretrained GNPS-based library and pipeline
    in which cleaning is the first data-preparation step
  repo: https://github.com/iomega/ms2query
- name: PubChem
  role: Source for adding missing InChIKeys by compound name and molecular formula
    lookup
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:d8094c2fbeb71dfd75bb1b5c4fa6340e4ada4516fe843796a6d7a59ad85af732
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MS2Query
  - PubChem
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# gnps-library-spectrum-preprocessing

## Summary

Clean and normalize GNPS library mass spectra with matchms before model training or library matching: filter low-quality spectra, remove noise peaks, cap peak mass, normalize intensities, and standardize metadata.

## When to use

Use when preparing a downloaded GNPS MGF library (e.g. ALL_GNPS.mgf) as training data or as a cleaned search library for MS2Deepscore/Spec2Vec/MS2Query, so that spectra are uniform and noise-free.

## When NOT to use

- Input is already a cleaned, intensity-normalized library such as the pretrained MS2Query sqlite library downloaded via 'ms2query --library ... --download' — re-cleaning is unnecessary and may alter peak lists.
- The task is peak picking or clustering of query MS2 spectra from LC-MS/MS runs; MS2Query itself does no peak picking, and tools like MZMine/FBMN should preprocess feature-level spectra first.

## Inputs

- GNPS library MGF file (e.g. ALL_GNPS.mgf or ALL_GNPS_NO_PROPOGATED.mgf downloaded from gnps-external.ucsd.edu)
- Per-spectrum peak lists and metadata (compound names, precursor m/z, ion mode, InChIKey where available)

## Outputs

- Cleaned, intensity-normalized, metadata-standardized spectral library split by ionization mode (matchms Spectrum objects / MGF)
- Library spectra annotated with InChIKeys (missing ones filled from PubChem) ready for structure-disjoint train/test splitting

## How to apply

Download the GNPS library (e.g. ALL_GNPS.mgf from gnps-external.ucsd.edu) and clean it with matchms: completely remove spectra with fewer than 3 peaks, remove peaks above 1000 Da, and remove peaks with intensity below 0.1% of the highest peak; for spectra with more than 500 peaks, remove the lowest-intensity peaks. Clean the metadata to a uniform format to remove or correct misplaced metadata, and normalize peak intensities; split into positive/negative ionization mode. Where structure annotations are needed (e.g. for structure-disjoint splits), add missing InChIKeys by searching compound name and molecular formula on PubChem via matchms extras. Record the cleaned library state before downstream model training so leakage checks can reference the original spectra.

## Related tools

- **matchms** (Performs library cleaning, metadata harmonization, peak filtering, intensity normalization, and InChIKey lookup via matchms extras) — github.com/iomega/ms2query
- **MS2Query** (Consumes the cleaned library; provides pretrained GNPS-based library and pipeline in which cleaning is the first data-preparation step) — https://github.com/iomega/ms2query
- **PubChem** (Source for adding missing InChIKeys by compound name and molecular formula lookup)

## Examples

```
ms2query --library ./library_folder --download --ionmode positive
```

## Evaluation signals

- No spectrum in the cleaned library has fewer than 3 peaks; spectra originally below this threshold are dropped entirely.
- All peak m/z values are ≤ 1000 Da and no peak has intensity < 0.1% of its spectrum's highest peak.
- Peak intensities are normalized (e.g. to a common scale) and spectra are partitioned into positive/negative ion-mode libraries.
- Metadata fields are in a uniform format with misplaced/corrupted entries removed or corrected; InChIKeys present where PubChem lookup succeeds.
- The cleaned library matches the version archived with the training-data-split manifest (e.g. Zenodo record 10.5281/zenodo.6811540) so train/test spectra can be checked against the original download.

## Limitations

- Cleaning is library-version dependent: results differ between GNPS snapshots (e.g. ALL_GNPS.mgf of 15 Nov 2021 vs ALL_GNPS_NO_PROPOGATED.mgf of 1 Nov 2022), so record the download date and URL.
- Peak filtering thresholds (1000 Da cap, 0.1% relative intensity, <3 peaks) were chosen for natural-product GNPS library benchmarking; other compound classes or instruments may need retuned cutoffs.
- Spectra lacking parseable compound names/formulas cannot have InChIKeys filled from PubChem and may be excluded from structure-disjoint splitting.

## Evidence

- [discussion] Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] Spectra with less than 3 peaks were completely removed from the library.: "Spectra with less than 3 peaks were completely removed from the library."
- [discussion] For spectra with more than 500 peaks, the peaks with the lowest intensities were removed.: "For spectra with more than 500 peaks, the peaks with the lowest intensities were removed."
- [discussion] The dataset was first cleaned using matchms33. The metadata was cleaned to get a uniform format and to remove or correct misplaced metadata.: "The dataset was first cleaned using matchms33. The metadata was cleaned to get a uniform format and to remove or correct misplaced metadata."
- [discussion] A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem.: "A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem."
