---
name: mass-spectrometry-library-matching
description: Use when you have a set of unannotated MS/MS consensus spectra (e.g.,
  an .mgf from MZmine or GNPS molecular networking) and need to identify which ones
  match metabolites from a specific taxonomic domain such as microbes.
when_to_use_negative:
- Input is already a feature table or abundance matrix with no MS/MS peak lists —
  library matching needs tandem spectra
- You need Level 1 identified compounds with standards rather than Level 2 GNPS-library
  or domain-attribution annotations
- Single-spectrum interactive searching is sufficient — use the standalone web apps
  (microbeMASST at masst.gnps2.org) instead of the batch jobs.py route
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: microbeMASST
  role: Primary domain-specific MASST library searched for taxonomically informed
    microbial metabolite matching
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py (microbe_masst)
  role: Batch driver script that submits spectra to the Fast Search API and writes
    _matches/_library/_datasets/_count_domain outputs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Backend endpoint (fasst.gnps2.org/fastsearch/) executing the spectral similarity
    searches against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and
    NORMAN data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Used to generate the input .mgf file of consensus MS/MS spectra
  repo: https://github.com/mzmine/mzmine
- name: GNPS_MASST
  role: Standalone web application code for single-spectrum searches against the MASSTs
  repo: https://github.com/mwang87/GNPS_MASST
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:025241190a5bb419f4d6facfd1aa128242557d7aa4f77c2a78b6f7e4c471d7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - GNPS_MASST
  - MZmine
  - jobs.py
  - jobs.py (microbe_masst)
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-spectrometry-library-matching

## Summary

Batch-search consensus MS/MS spectra (.mgf or USI lists) against taxonomically indexed reference libraries (microbeMASST and sibling domainMASSTs) via the GNPS Fast Search API to assign microbial, plant, tissue, or food origin to molecular features.

## When to use

Use when you have a set of unannotated MS/MS consensus spectra (e.g., an .mgf from MZmine or GNPS molecular networking) and need to identify which ones match metabolites from a specific taxonomic domain such as microbes.

## When NOT to use

- Input is already a feature table or abundance matrix with no MS/MS peak lists — library matching needs tandem spectra
- You need Level 1 identified compounds with standards rather than Level 2 GNPS-library or domain-attribution annotations
- Single-spectrum interactive searching is sufficient — use the standalone web apps (microbeMASST at masst.gnps2.org) instead of the batch jobs.py route

## Inputs

- Single .mgf file of consensus MS/MS spectra generated via MZmine or the GNPS molecular networking workflow
- List of USIs provided as a .csv or .tsv file
- jobs.py configuration entries mapping input files to output prefixes
- Dataset metadata (e.g., antibiotic-treated vs untreated sample groups) for post-hoc validation of matches

## Outputs

- _matches.tsv containing all scans matching each searched spectrum
- _library.tsv with matching GNPS library spectra (Level 2 MSI annotations)
- _datasets.tsv with unique matching sample counts per indexed dataset
- _count_domain.tsv files with per-domain MASST match counts
- Interactive HTML trees (_domain.html) and JSON tree files (e.g., _microbe.json)
- Filtered/aggregate .mgf files of domain-specific putative spectra

## How to apply

Prepare a single .mgf file (generated via MZmine or a GNPS molecular networking workflow) or a .csv/.tsv list of USIs, then add entries to jobs.py in the robinschmid/microbe_masst repository as ("input_directory/input_file", "output_directory/output_prefix"). Adjust the search parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — to match your research question or a published setting before running under Python 3.10. jobs.py leverages the Fast Search API (https://fasst.gnps2.org/fastsearch/) to batch-search all spectra against indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, producing _matches.tsv, _library.tsv, _datasets.tsv, interactive _domain.html trees, and per-domain _count_domain.tsv outputs. Because some Fast Search API calls fail intermittently, re-run jobs.py a couple of times with skip_existing=True until no new output is generated. Downstream, parse the _count_domain.tsv files to count domain-specific matches and apply filtering cascades (e.g., removing spectra that also match human cell lines) to isolate putative domain-of-origin spectra.

## Related tools

- **microbeMASST** (Primary domain-specific MASST library searched for taxonomically informed microbial metabolite matching) — https://masst.gnps2.org/microbemasst/
- **jobs.py (microbe_masst)** (Batch driver script that submits spectra to the Fast Search API and writes _matches/_library/_datasets/_count_domain outputs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Backend endpoint (fasst.gnps2.org/fastsearch/) executing the spectral similarity searches against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Used to generate the input .mgf file of consensus MS/MS spectra) — https://github.com/mzmine/mzmine
- **GNPS_MASST** (Standalone web application code for single-spectrum searches against the MASSTs) — https://github.com/mwang87/GNPS_MASST

## Examples

```
python code/jobs.py  # after adding ("input/spectra.mgf", "output/spf") and setting min cosine, mz tolerance, min matching peaks; re-run with skip_existing=True until no new output
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True until no new output is generated; a further run producing zero new matches confirms complete API coverage
- Reproduce known counts: in the SPF-mouse benchmark, 3,262 of 10,047 spectra had microbial matches, 837 human-cell-line matches were removed, leaving 2,425 (1,673 bacteria-only, 95 fungi-only, 657 both)
- Independent validation overlap: searching the 2,425 spectra against MSV000080918 should recover 621 overlaps with 512 exclusive to non-antibiotic-treated animals
- Confirm output schema: _matches.tsv, _library.tsv, _datasets.tsv, and _count_domain.tsv files exist for every input prefix and the _domain.html/_domain.json trees parse as valid HTML/JSON
- _library.tsv matches should correspond to GNPS library spectra, enabling Level 2 Metabolomics Standards Initiative annotations

## Limitations

- Some Fast Search API entries fail per run, so a single pass is insufficient — jobs.py must be re-run with skip_existing=True until outputs stabilize
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so matches can over-attribute without an additional filtering cascade (e.g., removing human-cell-line matches)
- Requires Python 3.10 specifically; search sensitivity depends on minimum cosine score, m/z tolerance, and minimum matching peaks settings chosen per research question
- No changelog is published for the tool/repository, so version drift must be tracked manually

## Evidence

- [methods] batch search via Fast Search API: "leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra"
- [methods] search parameters: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] re-run requirement: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] input formats: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
- [readme] output description: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed."
