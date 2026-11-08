---
name: taxonomy-based-spectrum-filtering
description: Use when a set of consensus MS/MS spectra (e.g., .mgf from MZmine molecular
  networking) must be labeled as putatively microbial and deconvoluted from host or
  human-cell-line spectral matches before downstream metabolomics interpretation.
when_to_use_negative:
- Input is a single spectrum for one-off annotation — use the standalone microbeMASST
  web app (https://masst.gnps2.org/microbemasst/) instead of the batch filtering cascade
- The goal is plant, food, or tissue attribution — those require plantMASST, foodMASST,
  or tissueMASST rather than the microbial filtering cascade
- Spectra lack a .mgf or USI (.csv/.tsv) representation and cannot be batch-searched
  via the Fast Search API
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: microbeMASST
  role: Taxonomically informed reference database searched to identify spectra with
    microbial matches
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch-search driver script that calls the Fast Search API over the input .mgf
    and writes _count_domain.tsv and other outputs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Backend search service executing spectrum queries against indexed GNPS/MassIVE,
    Metabolomics Workbench, Metabolights, and NORMAN data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Generates the input single .mgf consensus spectrum file from molecular networking
  repo: https://github.com/mzmine/mzmine
- name: Python 3.10
  role: Required runtime version for running jobs.py and the filtering cascade
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:238c467da9ea5f9ae53e666f7901b1fc9271547a02440ece25a0c4c25eafa17e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - MZmine
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# taxonomy-based-spectrum-filtering

## Summary

Filters a consensus MS/MS spectrum set by taxonomic origin using microbeMASST batch search results: spectra matching microbial references are retained, those also matching human cell lines are removed, and survivors are partitioned into bacteria-only, fungi-only, and both-kingdom classes.

## When to use

Use when a set of consensus MS/MS spectra (e.g., .mgf from MZmine molecular networking) must be labeled as putatively microbial and deconvoluted from host or human-cell-line spectral matches before downstream metabolomics interpretation.

## When NOT to use

- Input is a single spectrum for one-off annotation — use the standalone microbeMASST web app (https://masst.gnps2.org/microbemasst/) instead of the batch filtering cascade
- The goal is plant, food, or tissue attribution — those require plantMASST, foodMASST, or tissueMASST rather than the microbial filtering cascade
- Spectra lack a .mgf or USI (.csv/.tsv) representation and cannot be batch-searched via the Fast Search API

## Inputs

- .mgf file of consensus MS/MS spectra (e.g., 10,047 SPF-unique spectra generated via MZmine)
- microbeMASST batch search outputs: _count_domain.tsv, _matches.tsv, _datasets.tsv per spectrum
- human cell line (tissueMASST) match results used as a removal filter

## Outputs

- Filtered set of putative microbial spectra as a single aggregated .mgf (e.g., 2,425 spectra)
- Taxonomic partition counts: bacteria-only, fungi-only, and both-kingdom spectra (e.g., 1,673 / 95 / 657)
- Count of spectra removed due to human cell line matches (e.g., 837)

## How to apply

First run the microbeMASST batch search (jobs.py against the Fast Search API at https://fasst.gnps2.org/fastsearch/) on the .mgf file, setting minimum cosine score, m/z tolerance, and minimum matching peaks to the study's reported settings, and re-run with skip_existing=True until no new output appears because API calls can fail intermittently. Parse the _count_domain.tsv outputs to count spectra with microbial matches (e.g., 3,262 of 10,047 SPF-unique spectra) and identify those with additional human cell line matches (e.g., 837) that must be removed by the filtering cascade. The remaining putatively microbial spectra (e.g., 2,425) are split by taxonomic origin into bacteria-only, fungi-only, and both-kingdom bins, then aggregated into a single output .mgf. Validate by checking the cascade reproduces the expected counts exactly and the final .mgf contains the expected number of spectra.

## Related tools

- **microbeMASST** (Taxonomically informed reference database searched to identify spectra with microbial matches) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch-search driver script that calls the Fast Search API over the input .mgf and writes _count_domain.tsv and other outputs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Backend search service executing spectrum queries against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Generates the input single .mgf consensus spectrum file from molecular networking) — https://github.com/mzmine/mzmine
- **Python 3.10** (Required runtime version for running jobs.py and the filtering cascade) — https://github.com/robinschmid/microbe_masst

## Examples

```
python jobs.py  # with entries added as ("input_directory/input_file.mgf", "output_directory/output_prefix") and skip_existing=True; re-run until no new output
```

## Evaluation signals

- Filtering cascade reproduces exact expected counts: total microbial matches (e.g., 3,262), removed human-cell-line matches (e.g., 837), and remaining spectra (e.g., 2,425)
- Taxonomic partition sums to the filtered total: bacteria-only + fungi-only + both = remaining spectra (1,673 + 95 + 657 = 2,425)
- Aggregated output .mgf contains exactly the number of filtered spectra (2,425)
- Repeated jobs.py runs with skip_existing=True produce no new outputs, confirming all Fast Search API calls succeeded
- _count_domain.tsv files exist and parse for every searched spectrum

## Limitations

- Fast Search API entries can fail on a given run, requiring repeated jobs.py executions with skip_existing=True to catch all matches
- Python 3.10 is explicitly required; other versions may not work
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so raw match lists overstate domain coverage

## Evidence

- [results] filtering cascade yields: [evidence span withheld — anchor status: not_found]
- [readme] jobs.py re-run policy: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
- [readme] count_domain outputs: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
- [readme] search parameters: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] over-inclusive matches caveat: "This includes also samples that are not part of the curated domain-specific MASSTs."
