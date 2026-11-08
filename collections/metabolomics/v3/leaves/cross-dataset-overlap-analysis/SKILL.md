---
name: cross-dataset-overlap-analysis
description: Use when a list of candidate microbial MS/MS spectra (.mgf from MZmine/GNPS
  networking) must be cross-validated against an independent public dataset with treatment-vs-control
  metadata to test biological exclusivity.
when_to_use_negative:
- The spectra only need library-level annotation — use the _library.tsv GNPS library
  search output (MSI Level 2) instead of cross-dataset overlap analysis
- The spectra are raw unprocessed LC-MS/MS centroid files rather than a quantified
  MZmine-derived .mgf feature list
- The target dataset lacks treatment/group metadata, so spectra cannot be split into
  exclusive-vs-shared groups (microbeMASST matches alone cannot establish exclusivity)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Taxonomically informed domain-specific MASST backend searched during the batch
    run; its _count_domain.tsv outputs (e.g., _count_microbe.tsv) supply per-group
    match counts
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch-run driver that loops spectra through the Fast Search API and writes
    _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs for all domainMASSTs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Spectrum-matching endpoint queried by jobs.py against currently indexed GNPS/MassIVE,
    Metabolomics Workbench, Metabolights, and NORMAN data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Used upstream to generate the single .mgf file of putative microbial spectra
    that serves as batch-search input
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:1143b96c20eefe286133de6dcbff0f28925fc78cfba9d9e298ab00fe054fa2d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - jobs.py
  - microbeMASST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-dataset-overlap-analysis

## Summary

Batch-search a set of putative microbial MS/MS spectra (.mgf) into an independent public metabolomics dataset via microbeMASST/jobs.py, then quantify which spectra overlap the external dataset and which are exclusive to a metadata-defined control group, supporting microbial-origin claims.

## When to use

Use when a list of candidate microbial MS/MS spectra (.mgf from MZmine/GNPS networking) must be cross-validated against an independent public dataset with treatment-vs-control metadata to test biological exclusivity.

## When NOT to use

- The spectra only need library-level annotation — use the _library.tsv GNPS library search output (MSI Level 2) instead of cross-dataset overlap analysis
- The spectra are raw unprocessed LC-MS/MS centroid files rather than a quantified MZmine-derived .mgf feature list
- The target dataset lacks treatment/group metadata, so spectra cannot be split into exclusive-vs-shared groups (microbeMASST matches alone cannot establish exclusivity)

## Inputs

- MGF file of putative microbial MS/MS spectra (e.g., 2,425 spectra from the SPF mouse study, generated via MZmine/GNPS molecular networking)
- Target public dataset accession with sample metadata (e.g., MSV000080918: antibiotic-treated vs untreated animals)
- jobs.py script from robinschmid/microbe_masst with a files list of (input_file, output_prefix) entries

## Outputs

- Filtered _matches.tsv restricted to scans matching the target dataset accession
- Aggregated match counts per dataset and per sample group (from _datasets.tsv and _count_microbe.tsv)
- Summary .tsv overlap table (expected validation: 621 overlapping spectra, 512 exclusive to non-antibiotic animals)
- Subset .mgf file containing only the matched spectra

## How to apply

Load the .mgf of putative microbial spectra and the target dataset's metadata (e.g., MSV000080918 antibiotic-treated vs untreated groups) into pandas. Run jobs.py (robinschmid/microbe_masst) to batch-search all spectra against GNPS/MassIVE-indexed data through the Fast Search API, keeping default minimum cosine score, mz tolerance, and minimum matching peaks unless the question demands otherwise. Re-run jobs.py with skip_existing=True until no new output appears, since Fast Search API calls intermittently fail. Filter _matches.tsv to scans matching the target accession, then aggregate match counts per dataset and per sample group from _datasets.tsv and _count_microbe.tsv. Write the matched spectra subset as a .mgf and a summary .tsv; correctness is judged against the published validation: 621 overlapping spectra with 512 exclusive to non-antibiotic-treated animals.

## Related tools

- **microbeMASST** (Taxonomically informed domain-specific MASST backend searched during the batch run; its _count_domain.tsv outputs (e.g., _count_microbe.tsv) supply per-group match counts) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch-run driver that loops spectra through the Fast Search API and writes _matches.tsv, _datasets.tsv, and _count_domain.tsv outputs for all domainMASSTs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Spectrum-matching endpoint queried by jobs.py against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Used upstream to generate the single .mgf file of putative microbial spectra that serves as batch-search input) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py  # files list: ("input/putative_microbial.mgf", "out/msv000080918") with default cosine/mz/min-peaks, re-run until skip_existing=True yields nothing new
```

## Evaluation signals

- Summary table reproduces the published validation counts: 621 spectra overlapping MSV000080918 and 512 exclusive to animals not treated with antibiotics
- Every scan retained in the filtered _matches.tsv has a match in the target accession (MSV000080918) and appears in the written subset .mgf
- jobs.py has been re-run until a skip_existing=True pass produces no new output, confirming no spectra were silently dropped by transient Fast Search API failures
- Search parameters (minimum cosine score, mz tolerance, minimum matching peaks) were left at defaults and are recorded alongside the summary
- Aggregated counts from _datasets.tsv and _count_microbe.tsv agree: shared + group-exclusive spectra sum to the total overlap

## Limitations

- Fast Search API calls intermittently fail, so a single jobs.py run under-counts matches; repeated runs with skip_existing=True are required until output saturates
- _matches.tsv includes matching samples that are not part of the curated domain-specific MASSTs, so hits must be cross-checked against the domain counts before claiming microbial relevance
- Python 3.10 is required by jobs.py; other versions may break the batch run

## Evidence

- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest: "A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest"
- [readme] Make sure to run jobs.py a couple of times until no new output is generated with skip_existing=True: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
- [methods] A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum: "A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum in each currently indexed dataset."
- [other] Filter the _matches.tsv output to retain only scans with matches in MSV000080918.: [evidence span withheld — anchor status: not_found]
- [results] 621 spectra overlapped, of which 512 were exclusively present in animals not treated with antibiotics, supporting their microbial origin.: [evidence span withheld — anchor status: not_found]
