---
name: compound-to-taxon-mapping
description: Use when you have GNPS library or experimental MS/MS spectra (as .mgf
  entries or USIs) of candidate microbial metabolites and need to determine which
  microbial taxa they associate with, or whether they appear in non-microbial sources
  like human cell lines.
when_to_use_negative:
- Input is already a feature table or quantified abundance matrix rather than MS/MS
  spectra — microbeMASST requires searchable tandem spectra
- Query molecules are known plant, food, or tissue metabolites — the domain-specific
  plantMASST, tissueMASST, or foodMASST tools are the appropriate indices instead
  of microbeMASST
- Only single-spectrum ad hoc lookups are needed — the standalone microbeMASST web
  app searches one spectrum at a time without batch scripting
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: microbeMASST
  role: Taxonomically informed target index searched with query MS/MS spectra to retrieve
    microbial taxa matches
  repo: https://masst.gnps2.org/microbemasst/
- name: jobs.py
  role: Batch driver that calls the Fast Search API for multiple spectra and generates
    all per-domain output files
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: Fast Search API
  role: Underlying search endpoint executed by jobs.py against indexed public metabolomics
    data
  repo: https://fasst.gnps2.org/fastsearch/
- name: MZmine
  role: Prepares the .mgf query file for batch search
  repo: https://github.com/mzmine/mzmine
- name: GNPS libraries
  role: Source of reference library spectra used as query inputs and for _library.tsv
    Level 2 annotations
  repo: https://library.gnps2.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:f12e9fc7e10751bde96b3361632b7bdfb5ae5508b15e6d607f1f5561f7d6c5cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - GNPS libraries
  - MZmine
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# compound-to-taxon-mapping

## Summary

Maps MS/MS spectra of microbial metabolites to their producing taxa by batch-searching spectra against microbeMASST, a taxonomically informed search index of GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data, and aggregating the taxonomic distribution of matching samples.

## When to use

Use when you have GNPS library or experimental MS/MS spectra (as .mgf entries or USIs) of candidate microbial metabolites and need to determine which microbial taxa they associate with, or whether they appear in non-microbial sources like human cell lines.

## When NOT to use

- Input is already a feature table or quantified abundance matrix rather than MS/MS spectra — microbeMASST requires searchable tandem spectra
- Query molecules are known plant, food, or tissue metabolites — the domain-specific plantMASST, tissueMASST, or foodMASST tools are the appropriate indices instead of microbeMASST
- Only single-spectrum ad hoc lookups are needed — the standalone microbeMASST web app searches one spectrum at a time without batch scripting

## Inputs

- MS/MS spectra as a single .mgf file generated via MZmine or from GNPS molecular networking
- List of USIs provided as a .csv or .tsv file
- GNPS library reference spectra of known microbial molecules

## Outputs

- _matches.tsv listing all scans matching each searched spectrum
- _library.tsv listing matching GNPS library spectra (MSI Level 2 annotation support)
- _datasets.tsv with unique-sample match counts per indexed dataset
- _count_microbe.tsv with taxonomic match counts per domain MASST
- Interactive HTML tree (_microbe.html) and JSON (_microbe.json) of taxonomic hits
- Aggregated per-compound table of taxa and unique sample counts

## How to apply

Collect the query spectra (e.g., GNPS library spectra of lovastatin, salinosporamide A, commendamide, mevastatin, arylomycin A4, yersiniabactin, promicroferrioxamine, Glu-CA, Glu-DCA) as .mgf entries or a .csv/.tsv list of USIs. Run the batch search via jobs.py, which calls the Fast Search API against the currently indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, setting minimum cosine score, mz tolerance, and minimum matching peaks per the method defaults. Re-run jobs.py a couple of times with skip_existing=True until no new output is generated, since some Fast Search API entries can fail. Parse the _matches.tsv, _datasets.tsv, and _count_microbe.tsv files to extract per-compound taxonomic hits and unique sample counts, then aggregate into a structured table. Interpret taxonomic specificity, e.g., lovastatin matching uniquely to genus Aspergillus and salinosporamide A only to Salinispora tropica strains, with hits in human cell lines counting as evidence against microbial origin.

## Related tools

- **microbeMASST** (Taxonomically informed target index searched with query MS/MS spectra to retrieve microbial taxa matches) — https://masst.gnps2.org/microbemasst/
- **jobs.py** (Batch driver that calls the Fast Search API for multiple spectra and generates all per-domain output files) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **Fast Search API** (Underlying search endpoint executed by jobs.py against indexed public metabolomics data) — https://fasst.gnps2.org/fastsearch/
- **MZmine** (Prepares the .mgf query file for batch search) — https://github.com/mzmine/mzmine
- **GNPS libraries** (Source of reference library spectra used as query inputs and for _library.tsv Level 2 annotations) — https://library.gnps2.org/

## Examples

```
python code/jobs.py  # with files list entries ("input_directory/query.mgf", "output_directory/prefix") and skip_existing=True
```

## Evaluation signals

- Output TSV table exists and contains exactly nine compound rows for the nine-molecule validation set
- lovastatin shows Aspergillus as its exclusive taxon in the _count_microbe.tsv-derived table
- salinosporamide A maps only to Salinispora typica (Salinispora tropica) strains, not other taxa
- Re-running jobs.py with skip_existing=True produces no new output, confirming all matches were captured
- _matches.tsv, _datasets.tsv, and _count_microbe.tsv files are all generated for every queried spectrum

## Limitations

- Some Fast Search API entries fail on the first pass, requiring multiple re-runs with skip_existing=True until no new output is generated
- Search sensitivity depends on adjustable parameters (minimum cosine score, mz tolerance, number of minimum matching peaks) that must be tuned to the research question
- No changelog is available for the tool, so index coverage may change over time as GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN are re-indexed
- Python 3.10 must be used to run the batch workflow

## Evidence

- [methods] Fast Search API batch search against indexed public data: "execute a batch search of multiple MS/MS spectra against the current indexed data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN"
- [readme] Re-run requirement with skip_existing: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
- [methods] Adjustable search parameters: "Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks"
- [readme] Valid input formats: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
- [methods] microbeMASST taxonomic coverage: "microbeMASST | 8 | 20 | 48 | 124 | 278 | 561 | 1379 | 542"
