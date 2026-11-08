---
name: gzip-tsv-parsing
description: Use when a large .tsv.gz table is distributed via a Zenodo deposit (e.g.
  DOI 10.5281/zenodo.5607264) and must be loaded for row-level counting, deduplication,
  or validation of reported dataset totals.
when_to_use_negative:
- The file is plain uncompressed CSV/TSV with a different delimiter — pd.read_csv
  with compression='gzip' and sep='\t' would misparse it
- The question is a graph/query-level exploration of LOTUS entries rather than row-level
  counting of the flat table — SPARQL queries against the Wikidata hosting are the
  article's route for that
- Only aggregate statistics already published are needed, and the deposit table need
  not be downloaded or recomputed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the gzip-compressed TSV into a DataFrame (pd.read_csv with sep='\t',
    compression='gzip') and performs deduplication/counting of structure-organism
    pairs
- name: lotus-processor
  role: Source repository whose Zenodo deposit (10.5281/zenodo.5607264) contains the
    curated 3_curated/table.tsv.gz file to parse
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f73d658bee4793e42780f9d76d4510c83fa9f791502fe71de59f88e081e616eb
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - lotus-processor
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gzip-tsv-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Loads a gzip-compressed, tab-delimited table (e.g. the LOTUS lotus-processor curated deposit file) into a pandas DataFrame so that unique structure-organism pairs can be counted and cross-checked against reported dissemination figures.

## When to use

Use when a large .tsv.gz table is distributed via a Zenodo deposit (e.g. DOI 10.5281/zenodo.5607264) and must be loaded for row-level counting, deduplication, or validation of reported dataset totals.

## When NOT to use

- The file is plain uncompressed CSV/TSV with a different delimiter — pd.read_csv with compression='gzip' and sep='\t' would misparse it
- The question is a graph/query-level exploration of LOTUS entries rather than row-level counting of the flat table — SPARQL queries against the Wikidata hosting are the article's route for that
- Only aggregate statistics already published are needed, and the deposit table need not be downloaded or recomputed

## Inputs

- gzip-compressed TSV file: interim/tables/3_curated/table.tsv.gz from Zenodo deposit 10.5281/zenodo.5607264 (LOTUS curated structure-organism table)
- Reported target counts for validation: 588694 (3D pairs), 484174 (2D pairs), 750,000+ retained pairs

## Outputs

- pandas DataFrame of the curated LOTUS table
- structure_organism_pair_counts.json containing pair counts (3D and 2D), unique structure counts, and unique organism counts

## How to apply

Download the relevant Zenodo deposit and locate the target file, here interim/tables/3_curated/table.tsv.gz from the lotus-processor deposit (DOI 10.5281/zenodo.5607264). Load it directly without manual decompression using pandas: pd.read_csv(path, sep='\t', compression='gzip'), which transparently handles the gzip stream. Count unique referenced structure-organism pairs by dropping duplicate rows at two levels: the full stereochemical (3D) level and the flat (2D) level. As a cross-check, also tabulate unique curated structures and unique contributing organisms. Write the resulting counts (3D pairs, 2D pairs, structures, organisms) to a JSON file such as structure_organism_pair_counts.json, and validate against the reported figures of 588694 (3D) and 484174 (2D) pairs, whose sum is consistent with the reported 750,000+ retained pairs (~30% of 2.5M+ initial pairs).

## Related tools

- **pandas** (Loads the gzip-compressed TSV into a DataFrame (pd.read_csv with sep='\t', compression='gzip') and performs deduplication/counting of structure-organism pairs)
- **lotus-processor** (Source repository whose Zenodo deposit (10.5281/zenodo.5607264) contains the curated 3_curated/table.tsv.gz file to parse) — https://github.com/lotusnprod/lotus-processor

## Examples

```
df = pd.read_csv('interim/tables/3_curated/table.tsv.gz', sep='\t', compression='gzip')
```

## Evaluation signals

- Unique referenced structure-organism pair count at the 3D level must equal 588694
- Unique pair count at the 2D (flat) level must equal 484174
- Total retained pairs must be consistent with the reported 750,000+ (approximately 30% of the 2.5M+ initial pairs)
- Cross-check invariants: unique curated structure count and unique organism count must be plausible relative to the pair counts
- Output JSON (structure_organism_pair_counts.json) parses and contains the four required count fields

## Limitations

- Pair counts are level-dependent: the 3D (full stereochemistry) count (588694) and 2D (flat) count (484174) differ, so the aggregation level must be stated explicitly
- The article does not quantify which correct entries were removed or which incorrect ones kept by the LOTUS pipeline, so the parsed table inherits upstream curation errors
- The curated set is a filtered subset (~30% of initial pairs); results cannot be extrapolated to the raw 2.5M+ input pairs

## Evidence

- [other] pandas loading step: [evidence span withheld — anchor status: not_found]
- [other] validation thresholds: [evidence span withheld — anchor status: not_found]
