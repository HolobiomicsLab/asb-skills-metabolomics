---
name: public-massive-dataset-retrieval
description: 'Use when a benchmark requires a real public metabolomics dataset: the
  article reanalyzes the group1_B* MS2 files of MSV000096589 (drug-incubated bacterial
  cultures) to test library annotation gains.'
when_to_use_negative:
- The dataset is already processed into a feature table and MS2 spectra; this skill
  only covers raw-data retrieval.
- The experiment requires de novo data acquisition rather than reuse of the public
  drug-incubated bacterial culture dataset.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: ThermoRawFileParser
  role: converts Thermo RAW MS2 files from the public dataset into open mzML/MGF formats
    for downstream mzmine processing
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: mzmine
  role: processes the retrieved MS2 data files to generate the feature table and MS2
    spectra used for molecular networking and library annotation
  repo: https://github.com/mzmine/mzmine
- name: MassBank-data
  role: supplies open spectral records used alongside GNPS and MoNA when annotating
    the retrieved dataset's features
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:1aceac960520db6a7fccd14642c97ae1a3807cc3bf55777100346e0bc0e691e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CompOmics/ThermoRawFileParser
  tools:
  - MassBank-data
  - MassBank/MassBank-data
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# public-massive-dataset-retrieval

## Summary

Retrieve MS2 data files from a public MassIVE dataset (e.g. MSV000096589) so they can be processed in mzmine and evaluated against spectral library annotations, as done for MSnLib benchmarking.

## When to use

Use when a benchmark requires a real public metabolomics dataset: the article reanalyzes the group1_B* MS2 files of MSV000096589 (drug-incubated bacterial cultures) to test library annotation gains.

## When NOT to use

- The dataset is already processed into a feature table and MS2 spectra; this skill only covers raw-data retrieval.
- The experiment requires de novo data acquisition rather than reuse of the public drug-incubated bacterial culture dataset.

## Inputs

- MassIVE dataset accession (MSV000096589)
- group1_B* MS2 data files (Thermo RAW or open mzML/MGF)
- metadata sheet with unique sample id, plate_id and well/vial location

## Outputs

- Downloaded MS2 data files ready for mzmine processing
- Feature table and MS2 spectra for molecular networking
- Per-library annotation counts (129 open, 80 MSnLib, 150 total, 21 unique)

## How to apply

Locate the dataset accession on MassIVE (MSV000096589) and download the group1_B* MS2 data files that correspond to the drug-incubated bacterial culture experiment described in the article. Convert the raw Thermo files with ThermoRawFileParser if needed, or use the deposited open formats directly. Process the files in mzmine to build a feature table and MS2 spectra suitable for Feature-Based Molecular Networking and spectral library searching against GNPS, MoNA, MassBank EU and MSnLib reference entries. Finally, extract per-library annotation counts (open libraries 129, MSnLib 80, total 150, 21 unique MSnLib matches) and compare them to the reported values as a sanity check that the retrieval and processing chain reproduced the published result.

## Related tools

- **ThermoRawFileParser** (converts Thermo RAW MS2 files from the public dataset into open mzML/MGF formats for downstream mzmine processing) — https://github.com/CompOmics/ThermoRawFileParser
- **mzmine** (processes the retrieved MS2 data files to generate the feature table and MS2 spectra used for molecular networking and library annotation) — https://github.com/mzmine/mzmine
- **MassBank-data** (supplies open spectral records used alongside GNPS and MoNA when annotating the retrieved dataset's features) — https://github.com/MassBank/MassBank-data

## Examples

```
prefect worker start --pool local-work
```

## Evaluation signals

- The retrieved group1_B* files, when processed in mzmine and searched with FBMN, yield exactly 129 open-library annotations and 80 MSnLib annotations.
- Combining both sources reproduces the article's 150 total annotations with 21 unique MSnLib matches.
- All annotations can be traced back to the drug-incubated bacterial culture experiment described for MSV000096589.
- The metadata (unique sample id, plate_id, well/vial location) matches the template required by the MSnLib pipeline.

## Limitations

- The group1_B* subset is only part of MSV000096589; other batches may behave differently.
- Some external databases (e.g. DrugBank) need local files or special access and must be toggled in jobs.py before running the full annotation workflow.
- Annotation counts depend on the mzmine processing parameters; any deviation will change the reported numbers.

## Evidence

- [other] 1. Retrieve the group1_B* MS2 data files from the public MassIVE dataset MSV000096589.: [evidence span withheld — anchor status: not_found]
- [other] Open libraries yielded 129 annotations and MSnLib 80; MSnLib contributed 21 unique annotations, increasing the total to 150, of which 14 corresponded to added drugs.: [evidence span withheld — anchor status: not_found]
- [other] 2. Process the files in mzmine to generate a feature table and MS2 spectra for molecular networking.: [evidence span withheld — anchor status: not_found]
- [readme] Please use the [template]...for your metadata for having same column names and minimum needed information for the query.: "Please use the [template](https://docs.google.com/spreadsheets/d/1v6_IlGS3VgycGc-mSSdNeocY-CFXpONVZbuh3XNLX2E/edit?usp=sharing) for your metadata for having same column names and minimum needed"
