---
name: fast-search-api-query-execution
description: Use when you need to search one or more MS/MS spectra (a single spectrum
  or an .mgf file from MZmine or a USI .csv/.tsv list) against indexed GNPS/MassIVE,
  Metabolomics Workbench, Metabolights, and NORMAN data via the Fast Search API.
when_to_use_negative:
- When you want only a curated domain-specific search (e.g., only bacterial/fungal
  monoculture data) — the Fast Search hits all indexed public data including samples
  outside the curated microbeMASST domain, so post-filtering to the domain is required
- When processing raw LC-MS centroided data or building a feature table — mzmine,
  not the Fast Search API, is the tool for MS data processing; Fast Search expects
  already-derived MS/MS spectra in .mgf or USI form
- When you need deterministic single-run results — Fast Search API calls can fail
  silently, so a single jobs.py run cannot be treated as complete without re-runs
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Fast Search API
  role: Backend spectral search engine called by jobs.py to match query MS/MS spectra
    against all indexed public metabolomics data
  repo: https://fasst.gnps2.org/fastsearch/
- name: microbeMASST
  role: Domain-specific MASST web application and batch runner (jobs.py) that wraps
    the Fast Search API and produces taxonomically informed outputs
  repo: https://masst.gnps2.org/microbemasst/
- name: MZmine
  role: Source of .mgf files containing MS/MS spectra used as Fast Search API query
    input
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required runtime version for executing the jobs.py batch runner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:ac11b4cc28e5cf3c903b59eaf68820e1360b3841affa4611d6a078f11330f49c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fast-search-api-query-execution

## Summary

Executes a batch MS/MS spectral search against all currently indexed public datasets by calling the Fast Search API, which microbeMASST and the other domainMASSTs use as their spectral matching engine.

## When to use

Use when you need to search one or more MS/MS spectra (a single spectrum or an .mgf file from MZmine or a USI .csv/.tsv list) against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data via the Fast Search API.

## When NOT to use

- When you want only a curated domain-specific search (e.g., only bacterial/fungal monoculture data) — the Fast Search hits all indexed public data including samples outside the curated microbeMASST domain, so post-filtering to the domain is required
- When processing raw LC-MS centroided data or building a feature table — mzmine, not the Fast Search API, is the tool for MS data processing; Fast Search expects already-derived MS/MS spectra in .mgf or USI form
- When you need deterministic single-run results — Fast Search API calls can fail silently, so a single jobs.py run cannot be treated as complete without re-runs

## Inputs

- Single MS/MS spectrum or batch of spectra supplied as an .mgf file generated via MZmine
- List of Universal Spectrum Identifiers (USIs) supplied as a .csv or .tsv file
- jobs.py entries of the form ("input_directory/input_file", "output_directory/output_prefix")

## Outputs

- Interactive HTML trees per domain-specific MASST ending with _domain.html (e.g., _microbe.html)
- JSON files for the different trees (e.g., _microbe.json)
- _matches.tsv containing all scans matching the searched spectrum
- _library.tsv listing matching GNPS library spectra (Level 2 MSI annotation)
- _datasets.tsv with the number of unique matching samples per indexed dataset
- Series of _count_domain.tsv files with match counts per domain MASST

## How to apply

Load the query spectra into jobs.py either as a single MZmine-generated .mgf file or as a .csv/.tsv list of Universal Spectrum Identifiers, adding entries as ("input_directory/input_file", "output_directory/output_prefix"). Invoke the Fast Search API (https://fasst.gnps2.org/fastsearch/) so each spectrum is compared against all currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, tuning the minimum cosine score, precursor m/z tolerance, and minimum number of matching peaks to your research question. Because Fast Search API calls can silently fail for some entries, run jobs.py repeatedly with skip_existing=True until no new output is generated; Python 3.10 is required. The batch run simultaneously produces per-domainMASST outputs (_microbe.html trees, _matches.tsv, _library.tsv, _datasets.tsv, and _count_domain.tsv files), from which the microbeMASST-specific matches can be filtered downstream.

## Related tools

- **Fast Search API** (Backend spectral search engine called by jobs.py to match query MS/MS spectra against all indexed public metabolomics data) — https://fasst.gnps2.org/fastsearch/
- **microbeMASST** (Domain-specific MASST web application and batch runner (jobs.py) that wraps the Fast Search API and produces taxonomically informed outputs) — https://masst.gnps2.org/microbemasst/
- **MZmine** (Source of .mgf files containing MS/MS spectra used as Fast Search API query input) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required runtime version for executing the jobs.py batch runner)

## Examples

```
python code/jobs.py  # with entries like ("input/myspectra.mgf", "output/myquery"); re-run until no new output (skip_existing=True)
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True until no new output files are generated — a stable second run with no new _matches.tsv/_count_domain.tsv entries indicates all Fast Search API calls eventually succeeded
- Each of the six output types appears: _domain.html, _domain.json, _matches.tsv, _library.tsv, _datasets.tsv, and _count_domain.tsv per searched spectrum
- Every scan in _matches.tsv that is claimed as a microbeMASST hit traces back to a sample in a dataset indexed from GNPS/MassIVE, Metabolomics Workbench, Metabolights, or NORMAN
- Matching spectra in _library.tsv correspond to GNPS library spectra, enabling Level 2 Metabolomics Standards Initiative annotation
- Runtime environment is Python 3.10 (not a different major version), matching the README requirement

## Limitations

- Fast Search API calls can silently fail for individual entries; the README instructs running jobs.py a couple of times with skip_existing=True until no new output is generated (noted as 'should not be an issue anymore')
- Matches include samples outside the curated domain-specific MASSTs, so domain-restricted interpretation requires additional filtering
- Python 3.10 is explicitly required; other versions may not work
- Batch runner only accepts .mgf (via MZmine) or .csv/.tsv USI lists as input — raw centroided MS data must be preprocessed elsewhere

## Evidence

- [methods] Fast Search API is the backend spectral search called by jobs.py: [evidence span withheld — anchor status: not_found]
- [readme] jobs.py must be re-run until no new output appears: "Make sure to run jobs.py **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] Accepted input formats are MZmine .mgf or USI .csv/.tsv: [evidence span withheld — anchor status: not_found]
- [readme] Python 3.10 is required: "Please make user to use **_Python 3.10_**"
- [readme] Matches include samples outside the curated domain-specific MASSTs: "This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific"
- [readme] library.tsv enables Level 2 annotation via GNPS libraries: [evidence span withheld — anchor status: not_found]
- [methods] Search parameters: minimum cosine score, mz tolerance, minimum matching peaks: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
