---
name: natural-products-database-quality-verification
description: Use when a published database paper reports aggregate counts (e.g. retained
  structure-organism pairs, unique structures, organisms) and the deposited dataset
  (Zenodo deposit, TSV table) is available to reproduce them.
when_to_use_negative:
- The goal is to re-run the LOTUS processing/filtering pipeline itself (filter_dirty.R,
  MolVS sanitization) rather than verify counts on an already-curated table
- The deposited table is raw/uncurated, so counts would not correspond to the paper's
  reported curated totals
- The claim to verify is not a countable aggregate (e.g. qualitative validation rules
  or translation-quality judgments), which require manual expert review instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: lotus-processor
  role: Produces the curated LOTUS tables whose deposited outputs are downloaded and
    counted for verification
  repo: https://github.com/lotusnprod
- name: Zenodo deposit 10.5281/zenodo.5607264
  role: Archival source of the curated table.tsv.gz used as the verification input
  repo: https://doi.org/10.5281/zenodo.5607264
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:b92384bc95a7464454ca3bfbb4dd14ce421a44338639a0e1b4bd7fedfa25efe5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Zenodo deposit 10.5281/zenodo.5607264
  - lotus-processor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# natural-products-database-quality-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verifies reported quality/coverage figures of a curated natural-products database by independently recomputing record counts from the deposited dataset table, as done for LOTUS's 750,000+ referenced structure-organism pairs.

## When to use

Use when a published database paper reports aggregate counts (e.g. retained structure-organism pairs, unique structures, organisms) and the deposited dataset (Zenodo deposit, TSV table) is available to reproduce them.

## When NOT to use

- The goal is to re-run the LOTUS processing/filtering pipeline itself (filter_dirty.R, MolVS sanitization) rather than verify counts on an already-curated table
- The deposited table is raw/uncurated, so counts would not correspond to the paper's reported curated totals
- The claim to verify is not a countable aggregate (e.g. qualitative validation rules or translation-quality judgments), which require manual expert review instead

## Inputs

- gzip-compressed TSV of the curated LOTUS table (interim/tables/3_curated/table.tsv.gz) from Zenodo deposit DOI 10.5281/zenodo.5607264
- Published reported figures to validate against (e.g. 750,000+ retained pairs, 2.5M+ initial pairs, ~30% retention)

## Outputs

- structure_organism_pair_counts.json containing unique pair counts at 3D and 2D levels, unique structure counts, and unique organism counts
- Pass/fail comparison of computed counts against reported figures

## How to apply

Download the project's archival Zenodo deposit — for LOTUS this is DOI 10.5281/zenodo.5607264 — and locate the curated table, interim/tables/3_curated/table.tsv.gz. Load the gzip-compressed TSV into a pandas DataFrame with pd.read_csv(path, sep='\t', compression='gzip'), then deduplicate rows and count unique referenced structure-organism pairs twice: once at the full stereochemical (3D) level and once at the flat (2D) level. As a cross-check, also tabulate the number of unique curated chemical structures and unique contributing organisms in the table. Judge correctness by comparing computed counts against the paper's reported figures — LOTUS reports 588,694 (3D) and 484,174 (2D) unique pairs, with the retained set (~30%) consistent with the abstract's claim of 750,000+ pairs out of 2.5M+ initial ones. Persist the counts (pairs 3D/2D, structures, organisms) to a JSON summary file for auditability.

## Related tools

- **lotus-processor** (Produces the curated LOTUS tables whose deposited outputs are downloaded and counted for verification) — https://github.com/lotusnprod
- **Zenodo deposit 10.5281/zenodo.5607264** (Archival source of the curated table.tsv.gz used as the verification input) — https://doi.org/10.5281/zenodo.5607264

## Examples

```
import pandas as pd; df = pd.read_csv('interim/tables/3_curated/table.tsv.gz', sep='\t', compression='gzip'); pairs = df[['structure', 'organism']].drop_duplicates(); print(len(pairs))
```

## Evaluation signals

- Computed unique pair count at the stereochemical (3D) level equals the reported 588694
- Computed unique pair count at the flat (2D) level equals the reported 484174
- Overall retained total is consistent with 750,000+ pairs (~30% of 2.5M+ initial pairs)
- structure_organism_pair_counts.json is written with pair, structure, and organism counts and is internally consistent (2D pair count ≤ 3D pair count)

## Limitations

- Counting reproduces aggregate figures only; it does not re-validate individual entries (the paper's 420-pair manual validation and 100-entry spot check found 97% true positives but also that the pipeline 'removes a lot of correct entries and misses or induces some incorrect ones')
- Counts depend on the specific Zenodo deposit version; later deposits may shift figures relative to the published paper
- The 3D/2D distinction requires knowing which table columns encode stereochemistry; misinterpreting columns yields mismatched counts

## Evidence
