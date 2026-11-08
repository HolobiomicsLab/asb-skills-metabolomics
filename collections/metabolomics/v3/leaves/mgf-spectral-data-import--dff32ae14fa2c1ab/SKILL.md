---
name: mgf-spectral-data-import
description: Use when the input is a .mgf file of MS/MS feature spectra (unique feature
  identifier, precursor m/z, m/z–intensity tuples) that must be loaded into Python
  and preprocessed before specXplore similarity or dashboard work.
when_to_use_negative:
- Input is raw vendor MS data that has not yet been exported to .mgf — first process
  it with MZmine or vendor software to produce the .mgf export
- Input is already a processed feature table or similarity matrix with no MS/MS peak
  lists — this skill only imports spectra with m/z–intensity tuples
- Working on Windows for the current specXplore version — the README states the current
  version works on macOS and Linux but fails on Windows
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Imports the .mgf spectral data into Python and provides the Spectrum module
    for renaming/adding the 'feature_id' metadata key
  repo: https://matchms.readthedocs.io/en/latest/
- name: specXplore
  role: Provides the importing pipeline that applies basic specXplore data processing
    and auto-computes the three pairwise similarity scores
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Supplies the pretrained ms2deepscore model and library files required by the
    importing pipeline for machine-learning scores
- name: MZmine
  role: Upstream tool for exporting raw LC-MS/MS data into the .mgf format specXplore
    requires
  repo: https://mzmine.github.io/mzmine_documentation/getting_started.html
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:d9f0fae2bb9939e75d1e6a8808b62f0f2a7b6291902016bba19985129a1fc6ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2Query
  - MZmine
  - Python
  - matchms
  - plotly
  - spec2vec
  - specXplore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mgf-spectral-data-import

## Summary

Import LC-MS/MS feature spectra from an .mgf file into Python with matchms and apply basic specXplore preprocessing, ensuring every spectrum carries a unique 'feature_id' metadata key so downstream specXplore pairwise similarity computation (ms2deepscore, modified cosine, spec2vec) can run.

## When to use

Use when the input is a .mgf file of MS/MS feature spectra (unique feature identifier, precursor m/z, m/z–intensity tuples) that must be loaded into Python and preprocessed before specXplore similarity or dashboard work.

## When NOT to use

- Input is raw vendor MS data that has not yet been exported to .mgf — first process it with MZmine or vendor software to produce the .mgf export
- Input is already a processed feature table or similarity matrix with no MS/MS peak lists — this skill only imports spectra with m/z–intensity tuples
- Working on Windows for the current specXplore version — the README states the current version works on macOS and Linux but fails on Windows

## Inputs

- .mgf file with MS/MS feature spectra (unique feature identifier, precursor m/z, m/z–intensity tuples)
- MS2Query pretrained model and library files (positive mode: zenodo.org/records/10527997; negative mode: zenodo.org/records/10528030) placed in a 'pos'/'neg' models subdirectory

## Outputs

- matchms Spectrum objects in Python with feature_id, precursor m/z, and peak tuples
- specXplore session data object saved to the hard drive, containing the computed ms2deepscore, modified cosine, and spec2vec pairwise similarity matrices

## How to apply

Create a conda environment with python=3.8 and install specXplore, which pulls in matchms, jupyter-notebook, and the similarity backends. Load the .mgf file with matchms, verify each spectrum entry has precursor m/z and m/z–intensity tuples, and check that the feature identifier key is exactly 'feature_id' — if the source key differs, rename it via matchms.Spectrum metadata editing or a text-editor replacement of 'wrong_key=' with 'feature_id='. Raw vendor data must first be exported to .mgf (e.g. via MZmine3's GNPS/FBMN/MN export). Then run the specXplore importing pipeline, which applies basic specXplore data processing and automatically computes pairwise spectral similarities using ms2deepscore, modified cosine, and spec2vec; the pretrained ms2deepscore/MS2Query model and library files must be placed in a 'pos' (or negative-mode) models subdirectory matching the ionization mode. Rationale: this import step is mandatory — the article states data must be processed with the specXplore importing pipeline before the dashboard can be opened.

## Related tools

- **matchms** (Imports the .mgf spectral data into Python and provides the Spectrum module for renaming/adding the 'feature_id' metadata key) — https://matchms.readthedocs.io/en/latest/
- **specXplore** (Provides the importing pipeline that applies basic specXplore data processing and auto-computes the three pairwise similarity scores) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Supplies the pretrained ms2deepscore model and library files required by the importing pipeline for machine-learning scores)
- **MZmine** (Upstream tool for exporting raw LC-MS/MS data into the .mgf format specXplore requires) — https://mzmine.github.io/mzmine_documentation/getting_started.html

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- Every .mgf entry parses in matchms with a precursor m/z and m/z–intensity tuples, and each spectrum carries a unique 'feature_id' metadata key
- The specXplore importing pipeline runs to completion and emits a session data object containing all three pairwise similarity matrices (ms2deepscore, modified cosine, spec2vec)
- Runtime is consistent with reported benchmarks: wheat data (<1000 features) preprocesses in <15 min; urine data (<4000 features) in <75 min (~62 min MS2Query, ~9 min pairwise similarities)
- Installation is done inside a python=3.8 conda environment per the README; missing C++ compilers cause pip/Cython failures that must be resolved before import works
- Model and library files for the correct ionization mode are present in the models folder 'pos' (or negative-mode) subdirectory before the pipeline is invoked

## Limitations

- On macOS arm64 systems, ms2deepscore issue 199 (matchms/ms2deepscore#199) can yield similarity predictions inconsistent with other systems, silently affecting the default ms2deepscore scores computed by the importing pipeline
- The current specXplore version fails on Windows and requires ANACONDA plus Microsoft Visual C++ Redistributable workarounds
- Feature identifier key must be exactly 'feature_id'; any other key requires renaming via matchms.Spectrum metadata editing or a text editor before import
- No tool version numbers are stated for any of the software mentioned

## Evidence

- [methods] Import MS/MS spectral data in .mgf format using matchms: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [readme] Feature identifier key requirement: [evidence span withheld — anchor status: external_doc]
- [readme] .mgf format requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Automatic pairwise similarity computation during import: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] Preprocessing is mandatory before dashboard use: [evidence span withheld — anchor status: external_doc]
