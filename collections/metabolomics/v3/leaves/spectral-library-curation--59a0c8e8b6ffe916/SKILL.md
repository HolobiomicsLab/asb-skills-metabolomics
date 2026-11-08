---
name: spectral-library-curation
description: Use when a raw annotated spectral library (e.g. GNPS MGF exports) must
  be converted into a cleaned, harmonized peak-list dataset before training a spectral
  similarity model or embedding method.
when_to_use_negative:
- Spectra are already cleaned and binned by the downstream pipeline (re-filtering
  would double-transform intensities and change results)
- The target workflow needs negative ionisation mode spectra — the MS2DeepScore preprocessing
  described here discards them
- Only a handful of spectra are available; the curation choices (0.1% cutoff, 1000-peak
  cap) were tuned for large libraries (>100,000 spectra)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Python library used to import, filter (peak intensity/peak count/peak-number
    cutoffs), normalize (square-root transform) and write the GNPS spectra
  repo: https://github.com/matchms/ms2deepscore
- name: MS2DeepScore
  role: Downstream consumer of the cleaned spectra; the curation parameters (0.1%
    relative intensity, 1000-peak cap, sqrt transform) match its expected input preprocessing
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:d9e24bccf38de4131a1239225ac77cccb4a08ab0c46f21e9bbc5eab4fc4b6a3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spectral-library-curation

## Summary

Curate and clean annotated GNPS MS/MS library spectra (matchms) by removing low-quality entries, trimming low-intensity peaks, capping peak counts, and square-root transforming intensities to prepare a training/evaluation-ready spectral dataset (as done for MS2DeepScore).

## When to use

Use when a raw annotated spectral library (e.g. GNPS MGF exports) must be converted into a cleaned, harmonized peak-list dataset before training a spectral similarity model or embedding method.

## When NOT to use

- Spectra are already cleaned and binned by the downstream pipeline (re-filtering would double-transform intensities and change results)
- The target workflow needs negative ionisation mode spectra — the MS2DeepScore preprocessing described here discards them
- Only a handful of spectra are available; the curation choices (0.1% cutoff, 1000-peak cap) were tuned for large libraries (>100,000 spectra)

## Inputs

- Annotated GNPS MS/MS library spectra (MGF; Zenodo record 4699300, 210,407 spectra with InChIKey and SMILES and/or InChI)
- matchms spectrum objects with ionmode, peak m/z and intensity lists

## Outputs

- cleaned_spectra.mgf — cleaned peak lists with annotated metadata preserved
- Per-spectrum peak lists normalized to relative intensities with ≤1000 peaks and square-root-transformed intensities

## How to apply

Load the deposited annotated GNPS spectra (Zenodo record 4699300, 210,407 library entries retrieved 25/01/2021) with matchms. Discard spectra not in positive ionisation mode and those with fewer than 5 peaks between 10 and 1000 m/z. For each remaining spectrum, remove peaks with relative intensity below 0.1% of the maximum peak intensity and keep at most the 1000 highest-intensity peaks, then apply a square-root transformation to the peak intensities (this approximates the transformation applied by the downstream model, which bins peaks into 10,000 equal-width bins over 10–1000 m/z). Write the cleaned spectra to an MGF file (e.g. cleaned_spectra.mgf) with InChIKey/SMILES annotations preserved, since the downstream training split by unique InChIKeys (14,062/500/500) depends on intact structural metadata.

## Related tools

- **matchms** (Python library used to import, filter (peak intensity/peak count/peak-number cutoffs), normalize (square-root transform) and write the GNPS spectra) — https://github.com/matchms/ms2deepscore
- **MS2DeepScore** (Downstream consumer of the cleaned spectra; the curation parameters (0.1% relative intensity, 1000-peak cap, sqrt transform) match its expected input preprocessing) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.importing import load_from_mgf; spectra = load_from_mgf('gnps_library.mgf'); # filter: positive mode, >=5 peaks 10-1000 m/z, remove peaks <0.1% max intensity, cap 1000 peaks, sqrt intensities; save as MGF (cleaned_spectra.mgf)
```

## Evaluation signals

- All retained spectra are positive ionisation mode and contain ≥5 peaks between 10 and 1000 m/z
- Every cleaned spectrum has ≤1000 peaks and no peak with relative intensity <0.1% of its maximum peak intensity
- Peak intensities in cleaned_spectra.mgf are square-root transformed (distribution check: sqrt of original relative intensities)
- InChIKey/SMILES annotations survive the round-trip; counts reconcile with the 210,407 retrieved / 184,698 annotated spectra
- Downstream model training (109,734 training spectra) runs without import errors on the cleaned file

## Limitations

- Spectral libraries remain inherently biased toward certain compound classes and instrumentation types of varying quality, which the curation cannot correct
- Curation removes spectra rather than correcting them: entries not in positive mode or with too few peaks (fewer than 5 between 10 and 1000 m/z) are simply discarded
- The cleaned dataset is tied to the 25/01/2021 GNPS retrieval; a new retrieval or different parameters will yield a non-deterministic, non-identical cleaned set

## Evidence

- [discussion] Spectrum filtering: peaks <0.1% max intensity removed, max 1000 peaks, square-root transformed intensities: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Full cleaned dataset deposited on Zenodo: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
- [intro] Model trained on curated GNPS spectra: "The model was trained using a dataset of 109,734 MS/MS spectra, which was built through curating and cleaning spectra obtained from GNPS"
- [readme] README recommends matchms cleaning pipeline before training: [evidence span withheld — anchor status: external_doc]
