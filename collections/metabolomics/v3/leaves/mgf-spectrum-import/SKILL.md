---
name: mgf-spectrum-import
description: Use when a workflow starts from an MGF-formatted MS/MS file (e.g. all_your_spectrums.mgf
  or tests/testdata/pesticides.mgf) and spectra must be loaded into matchms Spectrum
  objects before applying default_filters or process_peaks pipelines.
when_to_use_negative:
- Input spectra are already parsed matchms Spectrum objects or already serialized
  to JSON — the import step is redundant and the workflow should begin at the filter/score
  stage.
- The task is similarity scoring on spectra already in memory; importing from MGF
  again adds no information and is not part of the score computation.
- Only a single spectrum's peak table is needed outside a matchms workflow — using
  pyteomics or pymzml directly may suffice rather than routing through matchms.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Provides matchms.importing.load_from_mgf for reading MGF files into Spectrum
    objects
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Backend used by matchms to parse MGF (and mzML/mzXML) files during import
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:a0bc1e06835173ececb481e63cb372b306ca46a2b845bb4f57ef06c94f66790d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mgf-spectrum-import

## Summary

Import tandem mass spectra from MGF files into matchms Spectrum objects, built on pyteomics, as the entry point for matchms metadata-cleaning, peak-filtering, and similarity-scoring workflows.

## When to use

Use when a workflow starts from an MGF-formatted MS/MS file (e.g. all_your_spectrums.mgf or tests/testdata/pesticides.mgf) and spectra must be loaded into matchms Spectrum objects before applying default_filters or process_peaks pipelines.

## When NOT to use

- Input spectra are already parsed matchms Spectrum objects or already serialized to JSON — the import step is redundant and the workflow should begin at the filter/score stage.
- The task is similarity scoring on spectra already in memory; importing from MGF again adds no information and is not part of the score computation.
- Only a single spectrum's peak table is needed outside a matchms workflow — using pyteomics or pymzml directly may suffice rather than routing through matchms.

## Inputs

- MGF-format tandem mass spectrometry (MS/MS) file containing multiple spectra
- Optionally other matchms-supported raw formats (msp, mzML, mzXML, JSON, metabolomics-USI)

## Outputs

- List of matchms Spectrum objects with parsed peaks and metadata
- Per-spectrum metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) available for downstream cleaning

## How to apply

Use matchms.importing.load_from_mgf to parse the MGF file into individual Spectrum objects; the paper notes matchms imports raw data from 'the commonly used formats msp, mzML, mzXML, MGF' and that the MGF importer is built on pyteomics. Iterate over the returned spectra and apply each downstream step per spectrum, e.g. spectrum = default_filters(spectrum) for metadata harmonization, or the paper's peak-processing stack select_by_mz(mz_from=0, mz_to=1000), select_by_relative_intensity(intensity_from=0.001), and require_minimum_number_of_peaks(n_required=10). Because metadata parsing in matchms is delegated to pyteomics/pymzml backends, verify that key fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) are present per spectrum before filtering. Each imported Spectrum is then a unit for subsequent cleaning or score computation (cosine, modified cosine). For reproducibility, matchms >= 0.18.0 offers a Pipeline class driven by a yaml workflow config that can run the same MGF import through cleaning and scoring in one call.

## Related tools

- **matchms** (Provides matchms.importing.load_from_mgf for reading MGF files into Spectrum objects) — https://github.com/matchms/matchms
- **pyteomics** (Backend used by matchms to parse MGF (and mzML/mzXML) files during import)

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; pipeline = Pipeline(create_workflow(score_computations=[["cosinegreedy", {"tolerance": 1.0}]])); pipeline.run("tests/testdata/pesticides.mgf")
```

## Evaluation signals

- Every spectrum in the input MGF file yields exactly one Spectrum object — e.g. the peak-count filter report lists each input spectrum exactly once.
- Imported Spectrum objects expose parseable metadata fields (ionmode, adduct, charge, InChIKey) that default_filters can clean without KeyError.
- Peaks loaded from the MGF retain m/z and intensity arrays usable by select_by_mz with the 0–1000 m/z window.
- When re-run via the matchms Pipeline with a yaml workflow config, the same MGF produces the same downstream filtered/scored results.

## Limitations

- matchms imports a range of formats (msp, mzML, mzXML, MGF, JSON, metabolomics-USI); format-specific quirks outside these are not covered.
- The paper notes large collected fragmentation datasets can contain inconsistent, wrongly labeled, or incorrect metadata — import alone does not fix this; default_filters must be applied afterwards.
- Version drift: matchms >= 0.18.0 shifted to a Pipeline class and sparse score handling, so older per-function invocation patterns may differ in recent releases.

## Evidence

- [results] MGF import path is pyteomics-backed: [evidence span withheld — anchor status: not_found]
- [abstract] Supported import formats: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] Per-spectrum filter application after import: "spectrum = default_filters(spectrum)"
- [abstract] Pyteomics as metadata-parsing backend: "metadata parsing can best be handled by using pyteomics or pymzml"
- [results] Concrete MGF input used in the paper's peak-processing example: [evidence span withheld — anchor status: not_found]
