---
name: mass-spectral-library-data-handling
description: Use when raw or public-library MS/MS spectra (mgf, msp, mzML, mzXML,
  JSON, USI) must be cleaned, peak-filtered, binned, and split by compound before
  training or benchmarking spectral similarity models.
when_to_use_negative:
- Input is already a cleaned feature table, embedding array, or molecular fingerprint
  matrix rather than raw/library MS/MS spectra files
- The task is model training hyperparameter tuning or prediction-uncertainty estimation
  on an already-prepared test set (e.g. the 3601-spectrum, 500-compound test set)
  rather than library data curation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python library for mass spectra import, handling, and comparisons; provides
    the recommended default cleaning/filtering pipeline
  repo: https://github.com/matchms/matchms
- name: MS2DeepScore
  role: Downstream consumer of cleaned, binned spectra for Siamese-network similarity
    prediction
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:7311cfef8b725d3abd409d22c5d087e299f290e3cc805ba830ba073eb75e4e3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectral-library-data-handling

## Summary

Import, clean, and prepare MS/MS spectral library data (e.g. GNPS-derived spectra) for downstream machine learning by applying matchms filtering pipelines, intensity/peak-count thresholds, m/z binning, and compound-level (InChIKey) train/validation/test splits.

## When to use

Use when raw or public-library MS/MS spectra (mgf, msp, mzML, mzXML, JSON, USI) must be cleaned, peak-filtered, binned, and split by compound before training or benchmarking spectral similarity models.

## When NOT to use

- Input is already a cleaned feature table, embedding array, or molecular fingerprint matrix rather than raw/library MS/MS spectra files
- The task is model training hyperparameter tuning or prediction-uncertainty estimation on an already-prepared test set (e.g. the 3601-spectrum, 500-compound test set) rather than library data curation

## Inputs

- Raw or library MS/MS spectra files in msp, mzML, mgf, mzXML, JSON, or USI format
- GNPS-annotated spectral library datasets (e.g. Zenodo record 4699300, 210,407 spectra)
- Compound annotations: InChIKey, SMILES, and/or InChI per spectrum

## Outputs

- Cleaned, peak-filtered spectra (peaks <0.1% max intensity removed, ≤1000 peaks, square-root transformed intensities)
- Binned spectrum vectors (10,000 equal-width bins, 10–1000 m/z)
- Compound-disjoint train/validation/test splits by InChIKey (e.g. 14,062/500/500)

## How to apply

Import spectra through matchms, which supports common formats (msp, mzml, mgf, mzxml, json, usi), and apply the default matchms cleaning pipeline before any model-facing step. Enforce the paper's filtering rules: remove peaks with intensities < 0.1% of the maximum peak intensity, keep at most the 1000 highest-intensity peaks, and square-root transform intensities. Bin peaks into 10,000 equally-sized bins spanning 10–1000 m/z to yield fixed-length vectors suitable for a Siamese network, then optionally augment (low-intensity peak removal, intensity jitter, new peak addition). Finally, split data by unique InChIKey (the paper used 14,062/500/500 training/validation/test) so that no compound appears in more than one split; the cleaned GNPS-derived dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) is available at Zenodo record 4699300 as a reference input.

## Related tools

- **matchms** (Python library for mass spectra import, handling, and comparisons; provides the recommended default cleaning/filtering pipeline) — https://github.com/matchms/matchms
- **MS2DeepScore** (Downstream consumer of cleaned, binned spectra for Siamese-network similarity prediction) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS)); pipeline.run("library_spectra.mgf")
```

## Evaluation signals

- Every cleaned spectrum has no peaks below 0.1% of its maximum peak intensity and contains at most 1000 peaks
- Binned representations have exactly 10,000 bins spanning 10–1000 m/z
- Train/validation/test splits are disjoint by InChIKey (no compound overlap; reference split 14,062/500/500)
- Annotated fraction check: cleaned GNPS dataset should retain InChIKey/SMILES annotations for the majority of spectra (reference: 184,698 of 210,407)
- All spectra load as matchms Spectrum objects regardless of source format (mgf, msp, mzML, etc.)

## Limitations

- Spectral libraries are inherently biased toward certain compound classes, which must be considered when applying models trained on them to experimental data
- Library spectra come from different instrumentation types, are of varying quality, and are unbalanced with regard to represented compound classes
- Only a subset of retrieved spectra are annotated (184,698 of 210,407 with InChIKey and SMILES and/or InChI), limiting supervised use to annotated entries

## Evidence

- [discussion] removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks.: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z.: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo"
- [results] MS2DeepScore is integrated in matchms, a recently developed Python library for mass spectra import, handling and comparisons: "MS2DeepScore is integrated in matchms, a recently developed Python library for mass spectra import, handling and comparisons"
- [readme] most common formats are supported, e.g. msp, mzml, mgf, mzxml, json, usi.: [evidence span withheld — anchor status: external_doc]
- [readme] Please first ensure cleaning your spectra. We recommend using the cleaning pipeline in [matchms](https://github.com/matchms/matchms).: [evidence span withheld — anchor status: external_doc]
