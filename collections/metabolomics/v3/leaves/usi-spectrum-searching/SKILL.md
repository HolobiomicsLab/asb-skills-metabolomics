---
name: usi-spectrum-searching
description: Use when you have one or a few MS/MS spectra (a single .mgf from MZmine
  or a USI list in .csv/.tsv) and want to identify which publicly deposited microbial
  monoculture datasets or GNPS library entries match those spectra.
when_to_use_negative:
- You need taxonomically restricted, curated microbeMASST sample counts only — the
  _matches.tsv output also contains samples outside the curated domain-specific MASSTs,
  so a separate curated-domain analysis is needed.
- Your input is raw LC-MS centroided data rather than an already deconvoluted .mgf
  or USI list — spectral preprocessing with MZmine is a prerequisite, not part of
  this search step.
- You cannot run Python 3.10 or repeatedly retry the search — the Fast Search API
  can silently drop entries, so a single incomplete run may leave some spectra unsearched.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: microbeMASST
  role: Standalone web application for searching one spectrum at a time against microbial
    monoculture data; batch searches use its microbe_masst jobs.py
  repo: https://masst.gnps2.org/microbemasst/
- name: Fast Search API
  role: Backend API invoked by jobs.py to batch-search MS/MS spectra against indexed
    GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data
  repo: https://fasst.gnps2.org/fastsearch/
- name: jobs.py
  role: Batch runner script that registers input files and parameters and drives Fast
    Search API calls for all domain-specific MASSTs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: MZmine
  role: Generates the single .mgf spectrum file that is submitted as search input
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:022f4f4a69846cb4e407b14a955706fd701c17448e2a0186076888a04dfc07db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# usi-spectrum-searching

## Summary

Search a user-supplied MS/MS spectrum, given as a USI-formatted .csv/.tsv or an MZmine .mgf file, against publicly indexed monoculture MS/MS data via the Fast Search API to find matching microbial datasets, samples, and GNPS library annotations.

## When to use

Use when you have one or a few MS/MS spectra (a single .mgf from MZmine or a USI list in .csv/.tsv) and want to identify which publicly deposited microbial monoculture datasets or GNPS library entries match those spectra.

## When NOT to use

- You need taxonomically restricted, curated microbeMASST sample counts only — the _matches.tsv output also contains samples outside the curated domain-specific MASSTs, so a separate curated-domain analysis is needed.
- Your input is raw LC-MS centroided data rather than an already deconvoluted .mgf or USI list — spectral preprocessing with MZmine is a prerequisite, not part of this search step.
- You cannot run Python 3.10 or repeatedly retry the search — the Fast Search API can silently drop entries, so a single incomplete run may leave some spectra unsearched.

## Inputs

- MS/MS spectra as a single .mgf file generated via MZmine from a GNPS molecular networking workflow
- List of Universal Spectrum Identifiers (USIs) in a .csv or .tsv file

## Outputs

- _matches.tsv listing all scans found to match each searched spectrum in the currently indexed data
- _library.tsv listing matched GNPS library spectra for Level 2 annotation
- _datasets.tsv with the number of unique matching samples per indexed dataset
- _count_domain.tsv files with matches for each specific domain MASST
- Interactive HTML trees (_domain.html) and JSON trees (_domain.json) per domain-specific MASST

## How to apply

Run jobs.py from robinschmid/microbe_masst (Python 3.10), adding one ("input_directory/input_file", "output_directory/output_prefix") entry per spectrum file. Invoke the Fast Search API at https://fasst.gnps2.org/fastsearch/ to batch-search the spectra against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data. Adjust the search parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — to your question, then re-run jobs.py several times with skip_existing=True until no new output is generated, because the Fast Search API can silently drop some entries. The run generates per-spectrum outputs: _matches.tsv (matching scans in indexed data, including samples outside the curated microbial domain), _library.tsv (matched GNPS library spectra enabling Level 2 Metabolomics Standards Initiative annotation), _datasets.tsv (number of unique matching samples per dataset), and _count_domain.tsv plus interactive HTML and JSON trees for each domain-specific MASST.

## Related tools

- **microbeMASST** (Standalone web application for searching one spectrum at a time against microbial monoculture data; batch searches use its microbe_masst jobs.py) — https://masst.gnps2.org/microbemasst/
- **Fast Search API** (Backend API invoked by jobs.py to batch-search MS/MS spectra against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data) — https://fasst.gnps2.org/fastsearch/
- **jobs.py** (Batch runner script that registers input files and parameters and drives Fast Search API calls for all domain-specific MASSTs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **MZmine** (Generates the single .mgf spectrum file that is submitted as search input) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py --input input/input.mgf --output output/microbe_masst --skip_existing True
```

## Evaluation signals

- Each searched spectrum yields a complete set of outputs: _matches.tsv, _library.tsv, _datasets.tsv, _count_domain.tsv, and one HTML plus JSON tree per domain-specific MASST.
- Re-run jobs.py with skip_existing=True repeatedly until a pass produces no new output files, confirming no spectra were silently skipped by the Fast Search API.
- _datasets.tsv should report counts of unique matching samples per dataset consistent with the _matches.tsv scan hits.
- _library.tsv hits should correspond to spectra from the GNPS libraries, enabling a Level 2 Metabolomics Standards Initiative annotation for the searched spectrum.
- The microbeMASST domain taxonomic coverage (8 kingdoms, 20 phyla, 48 classes, 124 orders, 278 families, 561 genera, 1379 species, 542 strains) is available in the lineages folder for cross-checking curated domain hits.

## Limitations

- The Fast Search API can fail for some entries, requiring repeated runs with skip_existing=True until all spectra are searched.
- The _matches.tsv output includes samples that are not part of the curated domain-specific MASSTs, so hits must be restricted to the microbial domain for taxonomically informed conclusions.
- Search result scope is limited to currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN; data not yet deposited are not searchable.
- Only one spectrum at a time can be searched via the standalone web applications; batch searching requires the jobs.py runner.

## Evidence

- [methods] Batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN"
- [methods] Adjustable search parameters: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] Output contents (matches and library hits): "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed."
- [readme] Library hits enable Level 2 annotation: "A _library.tsv file will be generated. This contains a list of spectra from the [GNPS libraries](https://library.gnps2.org/) found to match your spectrum of interest. This enables a Level 2"
- [readme] Input file formats: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of"
- [readme] Re-run requirement due to Fast Search API failures: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] Python version requirement: "Please make user to use **_Python 3.10_**"
