---
name: spectral-matching-parameter-tuning
description: Use when a set of extracted MS/MS spectra (.mgf from MZmine or a USI
  list) must be matched against GNPS/MassIVE-indexed data or domainMASSTs, and search
  sensitivity/precision must be controlled through cosine, m/z, and peak-matching
  thresholds.
when_to_use_negative:
- Query spectra are already curated library hits needing no parameter tuning — use
  a direct GNPS library search instead
- Input is a feature table or abundance matrix rather than MS/MS spectra (.mgf or
  USI list), which cannot be searched via the Fast Search API
- The goal is taxonomic visualization only with no re-searching — consume existing
  _count_domain.tsv/HTML trees rather than re-running jobs.py
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: microbeMASST
  role: Domain-specific MASST target of the parameterized spectral search against
    microbial metabolomics data
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch driver script that sets search parameters and calls the Fast Search
    API
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Backend endpoint executing the batch MS/MS spectral matching against indexed
    public data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Generates the single .mgf query file of MS/MS spectra
  repo: https://github.com/mzmine/mzmine
- name: GNPS_MASST
  role: Standalone web app for one-spectrum-at-a-time searches used to sanity-check
    tuned parameters
  repo: https://github.com/mwang87/GNPS_MASST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:1df2a7928ec980b8f593be30fe6046ddfda2f49d0297cd319fda82300dd22a52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - GNPS_MASST
  - MZmine
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-matching-parameter-tuning

## Summary

Tunes and applies MS/MS spectral search parameters (minimum cosine score, m/z tolerance, number of minimum matching peaks) when batch-searching spectra against public repositories and domain-specific MASST tools via the Fast Search API.

## When to use

Use when a set of extracted MS/MS spectra (.mgf from MZmine or a USI list) must be matched against GNPS/MassIVE-indexed data or domainMASSTs, and search sensitivity/precision must be controlled through cosine, m/z, and peak-matching thresholds.

## When NOT to use

- Query spectra are already curated library hits needing no parameter tuning — use a direct GNPS library search instead
- Input is a feature table or abundance matrix rather than MS/MS spectra (.mgf or USI list), which cannot be searched via the Fast Search API
- The goal is taxonomic visualization only with no re-searching — consume existing _count_domain.tsv/HTML trees rather than re-running jobs.py

## Inputs

- .mgf file of MS/MS spectra generated via MZmine (e.g., 2,425 putative microbial spectra)
- .csv or .tsv file of USIs as an alternative query format
- Dataset metadata distinguishing sample groups (e.g., antibiotic-treated vs untreated, MSV000080918)

## Outputs

- _matches.tsv containing all scans matching the searched spectra
- _library.tsv listing GNPS library spectra matches (MSI Level 2 annotations)
- _datasets.tsv with matching unique sample counts per indexed dataset
- _count_domain.tsv per-domain match counts and interactive HTML trees (_domain.html, e.g., _microbe.html)
- Filtered matched spectra .mgf and summary .tsv overlap table

## How to apply

Prepare the query spectra as a single .mgf file generated via MZmine (or a .csv/.tsv list of USIs), then run jobs.py from robinschmid/microbe_masst, adding entries as ('input_directory/input_file', 'output_directory/output_prefix'). Before running, check and adjust the search parameters — minimum cosine score, m/z tolerance, and number of minimum matching peaks — based on the research question; defaults were kept when reproducing the antibiotic-cocktail validation overlap (621 overlapping spectra of 2,425 putative microbial spectra against MSV000080918, 512 exclusive to untreated animals). Use Python 3.10 and the Fast Search API (https://fasst.gnps2.org/fastsearch/) so the batch search runs against currently indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data. Re-run jobs.py a couple of times with skip_existing=True until no new output is generated, because some Fast Search API entries fail intermittently and re-runs recover the missed matches. Validate parameter choices by checking that _matches.tsv retains only scans matching your spectra of interest and that downstream count tables (_datasets.tsv, _count_domain.tsv) reproduce expected overlap values.

## Related tools

- **microbeMASST** (Domain-specific MASST target of the parameterized spectral search against microbial metabolomics data) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch driver script that sets search parameters and calls the Fast Search API) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Backend endpoint executing the batch MS/MS spectral matching against indexed public data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Generates the single .mgf query file of MS/MS spectra) — https://github.com/mzmine/mzmine
- **GNPS_MASST** (Standalone web app for one-spectrum-at-a-time searches used to sanity-check tuned parameters) — https://github.com/mwang87/GNPS_MASST

## Examples

```
python code/jobs.py  # with entries ('input/microbe_spectra.mgf', 'output/antibiotic_overlap') and cosine/mz/min-peaks parameters checked; repeat with skip_existing=True until no new output
```

## Evaluation signals

- Summary overlap table reproduces the reported validation values: 621 overlapping spectra and 512 exclusive to non-antibiotic-treated animals out of 2,425 searched
- jobs.py re-run with skip_existing=True produces no new output files, confirming Fast Search API failures were recovered
- _matches.tsv contains only scans matching the searched query spectra from currently indexed datasets
- _datasets.tsv reports matching unique-sample counts consistent with _count_domain.tsv aggregates
- Parameter block explicitly records minimum cosine score, m/z tolerance, and number of minimum matching peaks used

## Limitations

- Fast Search API entries intermittently fail, so jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated
- Python 3.10 is explicitly required; other versions may break the workflow
- Only search targets in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN are indexed; matches are limited to currently indexed data and _matches.tsv includes samples outside curated domain-specific MASSTs

## Evidence

- [readme] Check and adjust the parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question."
- [readme] Re-run jobs.py until no new output due to Fast Search API failures.: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [methods] Leverage the Fast Search API for batch searches against indexed public repositories.: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN"
- [other] Default parameters reproduced the 621/512 antibiotic-validation overlap.: [evidence span withheld — anchor status: not_found]
