---
name: natural-product-database-inventory-counting
description: Use when verifying or reporting dataset-scale statistics for the NPDB
  scaffold release of G.A.M.E., e.g. validating that a downloaded deposit contains
  the expected number of curated natural products and scaffold index files.
when_to_use_negative:
- The goal is to run structure elucidation on a mass spectrum rather than to audit
  dataset contents — the csccp-solver-cli workflow is the appropriate path, not inventory
  counting
- Only the 26,641 test-suitable scaffolds (wmax = 500, 51,562 total configurations)
  are needed — those are a filtered subset, not the full 83,242 release inventory
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: G.A.M.E.
  role: Source of the NPDB and scaffold-file release whose inventory is being counted
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Scripting environment used to enumerate *.cIdx scaffold files and tally structure
    counts into the CSV manifest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:5f0953a3ec15c99ac979a864e2c6e67f9452601b204f58b6fad9f6ba29ece527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - G.A.M.E.
  - Python 2.7
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# natural-product-database-inventory-counting

## Summary

Reproduces the inventory counts of the natural products database (NPDB) deposited with G.A.M.E. by enumerating the 243,130 curated natural-product structures (sourced from DNP, the ZINC natural products subset, and the Traditional Chinese Medicine Database) and the 83,242 seed scaffold files (*.cIdx) generated from them.

## When to use

Use when verifying or reporting dataset-scale statistics for the NPDB scaffold release of G.A.M.E., e.g. validating that a downloaded deposit contains the expected number of curated natural products and scaffold index files.

## When NOT to use

- The goal is to run structure elucidation on a mass spectrum rather than to audit dataset contents — the csccp-solver-cli workflow is the appropriate path, not inventory counting
- Only the 26,641 test-suitable scaffolds (wmax = 500, 51,562 total configurations) are needed — those are a filtered subset, not the full 83,242 release inventory

## Inputs

- NPDB dataset files downloaded from the G.A.M.E. GitHub deposit or Zenodo mirror (https://doi.org/10.5281/zenodo.237579)
- Seed scaffold index files (*.cIdx), e.g. under GAME/examples/core_index
- Curated natural-product structure records in the NPDB source (from DNP, ZINC natural products subset, and Traditional Chinese Medicine Database)

## Outputs

- npdb_count_manifest.csv containing one row per count category: scaffold count (83,242) and natural-product total (243,130)

## How to apply

Download the NPDB dataset files from the G.A.M.E. deposit (github.com/CMDM-Lab/GAME, mirrored at https://doi.org/10.5281/zenodo.237579). Enumerate the seed scaffold index files (*.cIdx, e.g. under GAME/examples/core_index) using a Python 2.7 script and count them, expecting 83,242 scaffold files generated from the 243,130 collected natural products. Independently count the curated natural-product structures in the NPDB source, which were built by curating known structures from the Dictionary of Natural Products (DNP), the ZINC natural products subset, and the Traditional Chinese Medicine Database. Write both totals into a single CSV manifest (npdb_count_manifest.csv) with one row per count category, then validate that the scaffold-count row equals 83,242 and the natural-product total equals 243,130. Note the paper's distinction that a scaffold is the remaining core structure after all terminal side chains have been deleted, so the scaffold count is necessarily smaller than the structure count.

## Related tools

- **G.A.M.E.** (Source of the NPDB and scaffold-file release whose inventory is being counted) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Scripting environment used to enumerate *.cIdx scaffold files and tally structure counts into the CSV manifest)

## Examples

```
python2.7 -c "import glob; files = glob.glob('GAME/examples/core_index/*.cIdx'); print(len(files))  # expect 83242"
```

## Evaluation signals

- npdb_count_manifest.csv exists and contains a scaffold-count row equal to 83,242
- The manifest's natural-product total row equals 243,130
- The number of enumerated *.cIdx files under GAME/examples/core_index matches the scaffold count in the manifest
- The scaffold count (83,242) is strictly smaller than the curated structure count (243,130), consistent with scaffolds being cores after deletion of terminal side chains
- Structure provenance check: curated entries trace back to DNP, the ZINC natural products subset, and the Traditional Chinese Medicine Database

## Limitations

- The counts describe the full release inventory; only 26,641 scaffold files are suitable for testing with wmax = 500 (51,562 total configurations), so raw counts overestimate usable inputs
- Counting scripts require Python 2.7 (with numpy >= 1.7.1 and openbabel-python >= 1.3 for the surrounding GAME library), which is deprecated and may need adaptation
- The NPDB curation reflects the sources available to the 2017 study; counts will not match current releases of DNP, ZINC, or the Traditional Chinese Medicine Database

## Evidence

- [methods] 83,242 scaffold files generated from 243,130 natural products in the NPDB: "a total of 83,242 scaffold files were generated from the collected 243,130 natural products in the NPDB"
- [methods] NPDB provenance from DNP, ZINC natural products, and TCM Database: "a natural product database (NPDB) was built from curating all of the known structures from the Dictionary of Natural Products (DNP)"
- [methods] Scaffold definition as remaining core after side-chain deletion: "The scaffolds are the remaining core structures after all terminal side chains have been deleted."
- [readme] README location of seed scaffold files in the deposit: "All seed scaffolds (*.cIdx files) in our collected database that can be used as input of the "csccp-solver-cli" program were provided in the `GAME/examples/core_index`"
- [methods] Filtered subset suitable for testing: [evidence span withheld — anchor status: not_found]
