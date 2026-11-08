---
name: mgf-spectral-data-handling
description: Use when input spectra are provided as MGF files (e.g., experimental
  metabolomics runs or reference libraries) and must be converted into matchms spectrum
  objects for library-search scoring, filtering, or annotation counting.
when_to_use_negative:
- Input spectra are already in mzML, mzXML, msp, or JSON — use matchms's native readers
  for those formats directly rather than converting through MGF
- The goal is raw vendor-file conversion or peak picking upstream of library search;
  MGF handling here starts from already peak-picked MGF spectra
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Imports MGF files into Spectrum objects and applies metadata-cleaning and
    peak filters before spectral similarity comparison
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Downstream cosine score calculation on spectra loaded from MGF
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: spectraverse-analysis
  role: Project scaffold whose data/ directory receives the MGF files and whose scripts
    consume the loaded spectra
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:392f216f05bf1c4939e022dbc86783c714959b5b83926f8976f8954d0a3f546d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mgf-spectral-data-handling

## Summary

Loads experimental and reference MS/MS spectra stored in MGF format into matchms Spectrum objects so they can be cleaned, metadata-harmonized, and compared by spectral similarity. In the Spectraverse workflow, MGF files are the standard interchange format for both human blood metabolomics query spectra and the Spectraverse/MassSpecGym/NPLIB1 reference libraries.

## When to use

Use when input spectra are provided as MGF files (e.g., experimental metabolomics runs or reference libraries) and must be converted into matchms spectrum objects for library-search scoring, filtering, or annotation counting.

## When NOT to use

- Input spectra are already in mzML, mzXML, msp, or JSON — use matchms's native readers for those formats directly rather than converting through MGF
- The goal is raw vendor-file conversion or peak picking upstream of library search; MGF handling here starts from already peak-picked MGF spectra

## Inputs

- Experimental human blood metabolomics MS/MS spectra in MGF format
- Reference spectral libraries in MGF or matchms-readable format (Spectraverse v1.0.1 from Zenodo 10.5281/zenodo.17870921, MassSpecGym, NPLIB1)

## Outputs

- matchms spectrum objects loaded from each MGF file
- Cleaned/harmonized spectrum collections ready for cosine similarity library search

## How to apply

Place raw data files (MGF format) in the data/ directory of the spectraverse-analysis project, then use matchms to import each MGF file into Spectrum objects, since matchms natively supports MGF alongside mzML, mzXML, msp, and JSON. Apply matchms metadata cleaning and peak filters (e.g., library_cleaning_1.yaml / library_cleaning_2.yaml filter sets) after import so that precursor m/z, adduct, and fragment fields are harmonized before scoring. For the library-search comparison task, load the experimental human blood metabolomics MGF spectra and each reference library (Spectraverse v1.0.1 from Zenodo, MassSpecGym, NPLIB1) into separate spectrum-object collections so that the subsequent cosine similarity search with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions can be run identically against each library. Verify the import by checking that spectrum counts and precursor metadata match the source files before any filtering, so downstream annotation counts are attributable to search performance rather than ingestion errors.

## Related tools

- **matchms** (Imports MGF files into Spectrum objects and applies metadata-cleaning and peak filters before spectral similarity comparison) — https://github.com/matchms/matchms
- **SpectralEntropy** (Downstream cosine score calculation on spectra loaded from MGF) — https://github.com/YuanyueLi/SpectralEntropy
- **spectraverse-analysis** (Project scaffold whose data/ directory receives the MGF files and whose scripts consume the loaded spectra) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
git clone https://github.com/GuptaVishu2002/spectraverse.git && cp *.mgf spectraverse/data/ && conda activate spectraverse
```

## Evaluation signals

- All spectra from each MGF file are represented as matchms Spectrum objects (object count equals spectrum count in the file)
- Precursor m/z and adduct metadata are present and harmonized after matchms filter application (e.g., _get_neutral_mass handling of [M]+/[M]− adducts with reported charge of zero)
- The same loaded collection supports identical search settings (10 ppm precursor tolerance, minimum 3 matched fragment ions) across all three libraries
- Annotation-count table produced downstream covers all three libraries at all cosine similarity thresholds, indicating the MGF loading step did not silently drop spectra

## Limitations

- The methods section does not name specific dataset files or accession IDs for the MGF inputs; only the generic data/ directory convention is documented
- Script version numbers are not stated beyond the Python 3.10 requirement, so MGF-parsing behavior may vary across matchms versions
- matchms requires the modified matchms libraries bundled in the spectraverse-analysis repo (matchms/matchms.zip) to be substituted for the stock package, which may change default MGF parsing behavior

## Evidence

- [methods] Place raw data files (MGF format) in the `data/` directory.: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
- [other] Load the experimental human blood metabolomics MGF spectra and the three reference libraries (Spectraverse from Zenodo v1.0.1, MassSpecGym, NPLIB1) into matchms spectrum objects.: [evidence span withheld — anchor status: not_found]
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
- [readme] The software supports a range of popular spectral data formats, including mzML, mzXML, msp, metabolomics-USI, MGF, and JSON.: [evidence span withheld — anchor status: external_doc]
- [methods] use the zip file at matchms/matchms.zip and replace it original matchms libraries: [evidence span withheld — anchor status: not_found]
