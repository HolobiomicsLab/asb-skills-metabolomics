---
name: zenodo-dataset-retrieval
description: Use when a workflow needs the NPDB natural products database or seed
  scaffold files underlying the G.A.M.E. CSCCP solver, and the deposit contents must
  be enumerated and validated against published counts.
when_to_use_negative:
- The goal is to run structure elucidation on a single mass peak rather than curate
  or audit the deposited NPDB/scaffold corpus
- The dataset of interest is one of the four testing datasets (Cuscuta chinensis,
  Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) rather than the full
  NPDB scaffold database
- The deposit contents are needed only as generic Zenodo metadata; no enumeration
  or count validation of scaffold/NPDB files is required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: G.A.M.E.
  role: Source of the deposited datasets; the csccp-solver-cli consumes the enumerated
    *.cIdx seed scaffolds as input (-s) for CSCCP solving
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Scripting environment (with PyCuda 2017.1) used to enumerate and count the
    scaffold index files and write the count manifest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:bf2401598f245d87185ed40df231b5d6d1951450762d654f8a0b60c64765ff6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - G.A.M.E.
  - Python 2.7
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# zenodo-dataset-retrieval

## Summary

Retrieve and verify the G.A.M.E. natural products dataset deposited on Zenodo (doi 10.5281/zenodo.237579), confirming the curated NPDB contains 243,130 natural products and that 83,242 seed scaffold files (*.cIdx) were generated from them.

## When to use

Use when a workflow needs the NPDB natural products database or seed scaffold files underlying the G.A.M.E. CSCCP solver, and the deposit contents must be enumerated and validated against published counts.

## When NOT to use

- The goal is to run structure elucidation on a single mass peak rather than curate or audit the deposited NPDB/scaffold corpus
- The dataset of interest is one of the four testing datasets (Cuscuta chinensis, Ophiopogon japonicus, Polygonum multiflorum, Angelica sp.) rather than the full NPDB scaffold database
- The deposit contents are needed only as generic Zenodo metadata; no enumeration or count validation of scaffold/NPDB files is required

## Inputs

- Zenodo deposit archive at https://doi.org/10.5281/zenodo.237579 (G.A.M.E. datasets)
- GAME repository checkout (github.com/CMDM-Lab/GAME)
- Seed scaffold index files (*.cIdx) under GAME/examples/core_index
- NPDB natural-product structure source (DNP, ZINC natural products subset, Traditional Chinese Medicine Database)

## Outputs

- npdb_count_manifest.csv with rows for scaffold count (83,242) and natural-product total (243,130)
- Verified inventory of 83,242 *.cIdx seed scaffold files
- Filtered subset of 26,641 scaffolds suitable for wmax = 500 testing (51,562 total configurations)

## How to apply

Download the dataset files from the Zenodo deposit at https://doi.org/10.5281/zenodo.237579, which mirrors the GitHub repository (git clone https://github.com/CMDM-Lab/GAME.git). Enumerate the seed scaffold index files (*.cIdx) under GAME/examples/core_index and count them, expecting 83,242 scaffold files generated from the curated NPDB. Count the curated natural-product structures in the NPDB source, expecting 243,130 entries collected from the Dictionary of Natural Products (DNP), the ZINC natural products subset, and the Traditional Chinese Medicine Database. Record both totals in a manifest CSV with a row per count category, and note that only 26,641 scaffold files are suitable for testing with wmax = 500 (51,562 total configurations).

## Related tools

- **G.A.M.E.** (Source of the deposited datasets; the csccp-solver-cli consumes the enumerated *.cIdx seed scaffolds as input (-s) for CSCCP solving) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Scripting environment (with PyCuda 2017.1) used to enumerate and count the scaffold index files and write the count manifest)

## Examples

```
git clone https://github.com/CMDM-Lab/GAME.git && ls GAME/examples/core_index/*.cIdx | wc -l  # expect 83242
```

## Evaluation signals

- npdb_count_manifest.csv exists and the scaffold-count row equals 83,242
- The natural-product total row equals 243,130
- Number of *.cIdx files under GAME/examples/core_index matches 83,242
- Scaffolds passing the wmax = 500 filter number 26,641 with 51,562 total configurations
- Each scaffold's *.cIdx line 3 value x defines valid configuration indices (-c from 0 to x-1) per the README

## Limitations

- Only 26,641 of the 83,242 scaffold files are suitable for testing with wmax = 500, so a large majority of scaffolds are unusable for that test configuration
- The README dependencies (cuda 5, pycuda, numpy >= 1.7.1, openbabel-python >= 1.3, hadoop-1.0.3 for cluster support) predate the versions reported in the article (CUDA 7.5, PyCuda 2017.1), so environment setup may need reconciliation
- Structure data (SMILES, weights) are not displayed directly in scaffold query output and require parsing the underlying files

## Evidence

- [methods] a total of 83,242 scaffold files were generated from the collected 243,130 natural products in the NPDB: "a total of 83,242 scaffold files were generated from the collected 243,130 natural products in the NPDB"
- [methods] Zenodo DOI for all datasets: "All datasets are available on GitHub at https://doi.org/10.5281/zenodo.237579."
- [methods] NPDB curation sources: "a natural product database (NPDB) was built from curating all of the known structures from the Dictionary of Natural Products (DNP)"
- [methods] Only a subset of scaffolds passes the wmax test threshold: [evidence span withheld — anchor status: not_found]
- [readme] README location of seed scaffold files: "All seed scaffolds (*.cIdx files) in our collected database that can be used as input of the "csccp-solver-cli" program were provided in the `GAME/examples/core_index`"
