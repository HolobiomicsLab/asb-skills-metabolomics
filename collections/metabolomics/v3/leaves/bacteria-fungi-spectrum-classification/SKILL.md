---
name: bacteria-fungi-spectrum-classification
description: Use when a consensus .mgf of spectra has been batch-searched against
  microbeMASST and you need to (a) subtract spectra with non-microbial (e.g., human
  cell line) matches and (b) split the remaining putative microbial spectra by taxonomic
  origin into bacteria, fungi, or both.
when_to_use_negative:
- Input is a feature table or annotation table rather than consensus MS/MS spectra
  in .mgf or USI list format — the batch search requires spectra from MZmine molecular
  networking or a .csv/.tsv of USIs
- The goal is molecular-level annotation against GNPS spectral libraries (a _library.tsv
  / MSI Level 2 annotation task), not taxonomic attribution of spectra to microbial
  domains
- Single-spectrum interactive lookup — use the standalone microbeMASST web app rather
  than the batch jobs.py cascade
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Domain-specific MASST reference providing the microbial (bacterial and fungal)
    matches used for taxonomic classification
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch search script that queries the Fast Search API for all spectra against
    indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data and
    emits _count_domain.tsv outputs consumed by the classification cascade
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Backend search endpoint used by jobs.py to match each consensus spectrum against
    the indexed domainMASST data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Tool used to generate the single .mgf consensus spectrum file that serves
    as batch-search input
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required runtime for executing jobs.py and for parsing/filtering the _count_domain.tsv
    outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:e46a5a9d84a5252f1497acd428541e40ee30ab8d3c3fff0d7910e4729a11a1eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bacteria-fungi-spectrum-classification

## Summary

Taxonomic partitioning of MS/MS spectra into bacteria-only, fungi-only, and bacteria+fungi categories after removing spectra that also match human (or other non-microbial) reference data. This turns raw microbeMASST match counts into a defensible set of putative microbial spectra attributable to specific microbial domains.

## When to use

Use when a consensus .mgf of spectra has been batch-searched against microbeMASST and you need to (a) subtract spectra with non-microbial (e.g., human cell line) matches and (b) split the remaining putative microbial spectra by taxonomic origin into bacteria, fungi, or both.

## When NOT to use

- Input is a feature table or annotation table rather than consensus MS/MS spectra in .mgf or USI list format — the batch search requires spectra from MZmine molecular networking or a .csv/.tsv of USIs
- The goal is molecular-level annotation against GNPS spectral libraries (a _library.tsv / MSI Level 2 annotation task), not taxonomic attribution of spectra to microbial domains
- Single-spectrum interactive lookup — use the standalone microbeMASST web app rather than the batch jobs.py cascade

## Inputs

- Consensus MS/MS spectra file (.mgf generated via MZmine, e.g., 10,047 SPF-unique spectra)
- _count_domain.tsv outputs from microbeMASST batch search (per-domain match counts per scan)
- _matches.tsv files listing scans and their indexed matches
- Non-microbial reference search results (e.g., human cell line / tissueMASST-style matches to subtract)

## Outputs

- Filtered set of putative microbial spectra (e.g., 2,425 spectra)
- Taxonomic partition counts: bacteria-only (1,673), fungi-only (95), bacteria and fungi (657)
- Aggregated output .mgf file containing the 2,425 filtered spectra

## How to apply

After running jobs.py (with Python 3.10) against the Fast Search API (https://fasst.gnps2.org/fastsearch/) using the paper's parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — parse the _count_domain.tsv outputs to count spectra with microbial matches. Apply the filtering cascade: remove spectra that also match a non-microbial domain reference (e.g., 837 human-cell-line matches removed from 3,262 microbial matches, leaving 2,425). Then classify the remaining spectra by their domain-level microbeMASST hits into bacteria-only, fungi-only, or bacteria+fungi (e.g., 1,673 / 95 / 657). Finally aggregate the filtered spectra into a single output .mgf. Re-run jobs.py with skip_existing=True multiple times until no new output appears, since some Fast Search API calls fail and sequential re-runs recover the remaining matches.

## Related tools

- **microbeMASST** (Domain-specific MASST reference providing the microbial (bacterial and fungal) matches used for taxonomic classification) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch search script that queries the Fast Search API for all spectra against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data and emits _count_domain.tsv outputs consumed by the classification cascade) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Backend search endpoint used by jobs.py to match each consensus spectrum against the indexed domainMASST data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Tool used to generate the single .mgf consensus spectrum file that serves as batch-search input) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required runtime for executing jobs.py and for parsing/filtering the _count_domain.tsv outputs)

## Examples

```
python code/jobs.py  # with files list: ("input/10k_spf_unique.mgf", "output/spf"), skip_existing=True; re-run until no new _count_domain.tsv output
```

## Evaluation signals

- Filtering cascade reproduces the reported counts exactly: 3,262 total microbial matches, 837 removed as human-cell-line matches, 2,425 remaining
- Taxonomic split of remaining spectra sums correctly: 1,673 bacteria-only + 95 fungi-only + 657 both = 2,425
- Aggregated output .mgf contains exactly 2,425 spectra
- jobs.py re-run with skip_existing=True produces no new output (all spectra searched successfully)
- Every spectrum counted as putative microbial has at least one domain-level match in a _count_domain.tsv file and no conflicting non-microbial domain match

## Limitations

- The Fast Search API intermittently fails entries; jobs.py must be re-run multiple times with skip_existing=True until no new output is generated, otherwise match counts are under-counted
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so subtraction against an appropriate non-microbial reference (e.g., human cell lines) is required before attributing spectra to microbes
- Search parameters (minimum cosine score, m/z tolerance, minimum matching peaks) must match the paper's reported settings or counts will not be comparable

## Evidence

- [readme] A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST.: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True.: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific MASSTs.: "This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific"
- [results] Of 3,262 spectra with microbial matches, 837 also matched human cell lines and were removed, leaving 2,425 putative microbial spectra: 1,673 exclusive to bacteria, 95 to fungi, and 657 in both.: [evidence span withheld — anchor status: not_found]
- [methods] leverage the Fast Search API and execute a batch search of multiple MS/MS spectra: "leverage the [Fast Search API](https://fasst.gnps2.org/fastsearch/) and execute a batch search of multiple MS/MS spectra"
