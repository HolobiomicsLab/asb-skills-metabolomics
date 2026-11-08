---
name: mass-spectrometry-peak-list-processing
description: Use when raw MS/MS spectra (mzML, mzXML, msp, MGF, JSON, or metabolomics-USI)
  contain inconsistent metadata, noisy peaks, or incompatible intensity scales and
  must be cleaned, filtered, and normalized before similarity scoring.
when_to_use_negative:
- Input spectra are already normalized and cleaned, e.g. from a library that went
  through the LIBRARY_CLEANING default pipeline in matchms >= 0.18.0
- The goal is spectral visualization rather than peak-list processing — the README
  points to spectrum_utils as the lighter-weight, visualization-focused package
- Raw data are chromatograms or profile-level MS1 rather than imported peak lists;
  matchms operates on imported MS/MS peak spectra
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: 'Primary Python package: imports MS/MS spectra, applies stacked filter functions
    (normalize_intensities, select_by_mz, default_filters) and builds the processing
    pipeline'
  repo: https://github.com/matchms/matchms
- name: pyteomics
  role: Backend for mzML, mzXML, and MGF file importers used to read raw spectra into
    the pipeline
- name: pymzml
  role: Alternative backend for metadata parsing of mzML files
- name: spectrum_utils
  role: Lightweight Python package with a focus on spectra visualization, mentioned
    as the complementary (non-processing) option
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:96df6dca72fe5d5344c92dc7b8fd9b39c0526644254ca0318e5ff5562ab853b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pymzml
  - pyteomics
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mass-spectrometry-peak-list-processing

## Summary

Processing of MS/MS peak lists with matchms: importing spectra from msp, mzML, mzXML, MGF, JSON, or USI, stacking metadata-cleaning and peak filter functions (e.g. select_by_mz, select_by_relative_intensity, require_minimum_number_of_peaks, normalize_intensities) into a reproducible pipeline.

## When to use

Use when raw MS/MS spectra (mzML, mzXML, msp, MGF, JSON, or metabolomics-USI) contain inconsistent metadata, noisy peaks, or incompatible intensity scales and must be cleaned, filtered, and normalized before similarity scoring.

## When NOT to use

- Input spectra are already normalized and cleaned, e.g. from a library that went through the LIBRARY_CLEANING default pipeline in matchms >= 0.18.0
- The goal is spectral visualization rather than peak-list processing — the README points to spectrum_utils as the lighter-weight, visualization-focused package
- Raw data are chromatograms or profile-level MS1 rather than imported peak lists; matchms operates on imported MS/MS peak spectra

## Inputs

- MS/MS spectra in msp, mzML, mzXML, MGF, JSON, or metabolomics-USI format (e.g. tests/testdata/pesticides.mgf)
- Spectrum metadata fields: compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge

## Outputs

- Filtered spectra with cleaned, harmonized metadata
- Normalized peak lists (m/z and intensity pairs) with maximum intensity of 1
- Spectra ready as references and queries for cosine / modified cosine similarity scoring
- Optionally, a yaml workflow config produced by create_workflow for pipeline reproducibility

## How to apply

Build a pipeline with matchms: load spectra with matchms.load_from_mgf (or importers for msp/mzML/mzXML/JSON/USI), then stack filter functions — matchms.filtering.default_filters for baseline metadata cleaning, followed by normalize_intensities to rescale each spectrum's peak intensities to a maximum of 1. For per-spectrum peak tuning, apply select_by_mz (e.g. mz_from=0, mz_to=1000), select_by_relative_intensity (e.g. intensity_from=0.001), and require_minimum_number_of_peaks (n_required=10). The normalized, filtered peak lists (m/z and intensity pairs) then serve as references and queries for Cosine Greedy or modified cosine score calculation. From matchms >= 0.18.0 the Pipeline class can encode this workflow in a yaml file for reproducibility. Validate by checking that each output spectrum has a non-empty peak list and that its maximum peak intensity equals 1.

## Related tools

- **matchms** (Primary Python package: imports MS/MS spectra, applies stacked filter functions (normalize_intensities, select_by_mz, default_filters) and builds the processing pipeline) — https://github.com/matchms/matchms
- **pyteomics** (Backend for mzML, mzXML, and MGF file importers used to read raw spectra into the pipeline)
- **pymzml** (Alternative backend for metadata parsing of mzML files)
- **spectrum_utils** (Lightweight Python package with a focus on spectra visualization, mentioned as the complementary (non-processing) option)

## Examples

```
workflow = create_workflow(yaml_file_name="my_config_file.yaml", score_computations=[["cosinegreedy", {"tolerance": 1.0}]]); pipeline = Pipeline(workflow); pipeline.run("tests/testdata/pesticides.mgf")
```

## Evaluation signals

- For every filtered spectrum, maximum peak intensity equals 1 after normalize_intensities
- Each output spectrum retains a non-empty peak list (require_minimum_number_of_peaks with n_required=10 guards emptiness)
- Metadata fields (compound name, InChI, SMILES, InChIKey, ionmode, adduct, charge) are present, cleaned, and harmonized per the applied filters
- All spectra that enter Cosine Greedy / modified cosine score computation have gone through the same filter stack, so reference and query peak lists are comparable
- When Pipeline is used, the stored yaml config re-runs the workflow and reproduces the same filtered peak lists

## Limitations

- Large collected fragmentation MS/MS datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata, which filter stacks must address
- Spectral comparison speed and sparse score storage considerations mean not all pairwise scores need to be computed or stored (matchms >= 0.18.0)
- matchms ecosystem packages (spec2vec, ms2deepscore, ms2query) have NumPy-version-dependent compatibility; ms2query was flagged incompatible in the README matrix

## Evidence

- [abstract] Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] Available filters include extensive cleaning, correcting, checking of key metadata fields such as compound name, structure annotations (InChI, SMILES, InChIKey), ionmode, adduct, or charge: [evidence span withheld — anchor status: not_found]
- [abstract] It allows to implement and run an easy-to-follow, easy-to-reproduce workflow from raw mass spectra to pre- and post-processed spectral data: "It allows to implement and run an easy-to-follow, easy-to-reproduce workflow from raw mass spectra to pre- and post-processed spectral data"
- [readme] To make typical matchms workflows (data import, processing, score computations) more accessible to users, matchms now offers a Pipeline class to handle complex workflows: [evidence span withheld — anchor status: not_found]
- [abstract] select_by_relative_intensity example: "select_by_relative_intensity(s, intensity_from=0.001)"
- [readme] README advanced pipeline example with DEFAULT_FILTERS and LIBRARY_CLEANING: "query_filters=DEFAULT_FILTERS,
        reference_filters=LIBRARY_CLEANING + ["add_fingerprint"]"
