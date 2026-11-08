---
name: corpus-statistics-reproduction
description: Use when a task asks to reproduce or verify published corpus-level counts
  for the PoDP Zenodo deposit — e.g. 'how many metagenomes, genomes, and MAGs among
  the >2,600 genomic sources?' — by recomputing them from the archived JSON project
  metadata.
when_to_use_negative:
- The goal involves the raw metabolomics or genomics data themselves (spectra, genome
  sequences) — the PoDP captures only metadata that pairs omics datasets in external
  repositories such as MassIVE, MetaboLights, and GenBank
- The task requires per-project curation, submission, or search/filtering in the PoDP
  web application rather than corpus-wide aggregate counts
- The deposit under analysis is a live platform export mid-cycle; counts will drift
  between the monthly Zenodo uploads, so verify against the archived snapshot cited
  by the paper
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project metadata (pairs of omics datasets
    with modular metadata fields) whose JSON export is counted
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Archive from which the PoDP project JSON files are downloaded for the reproducibility
    computation (doi:10.5281/zenodo.6091663)
- name: Python
  role: Parser/aggregator for the JSON project documents used to compute all Box 1
    counts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:6599eec09b62965cd8645886270ca7e2ece93396140579fe3f26fcf758ccb219
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Python
  - Zenodo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# corpus-statistics-reproduction

## Summary

Reproduce the PoDP corpus-level dataset statistics (Box 1 counts) by parsing the community-curated project metadata JSON files archived on Zenodo and aggregating counts of projects, MS samples, genomic source types, modular metadata methods, and BGC–MS/MS links.

## When to use

Use when a task asks to reproduce or verify published corpus-level counts for the PoDP Zenodo deposit — e.g. 'how many metagenomes, genomes, and MAGs among the >2,600 genomic sources?' — by recomputing them from the archived JSON project metadata.

## When NOT to use

- The goal involves the raw metabolomics or genomics data themselves (spectra, genome sequences) — the PoDP captures only metadata that pairs omics datasets in external repositories such as MassIVE, MetaboLights, and GenBank
- The task requires per-project curation, submission, or search/filtering in the PoDP web application rather than corpus-wide aggregate counts
- The deposit under analysis is a live platform export mid-cycle; counts will drift between the monthly Zenodo uploads, so verify against the archived snapshot cited by the paper

## Inputs

- PoDP Zenodo deposit of community-curated project metadata as JSON documents (doi:10.5281/zenodo.6091663)
- JSON schema describing the project format (app/public/schema.json from the paired-data-form repository)

## Outputs

- statistics_table.csv with counts matching Box 1: 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 metagenome-assembled genomes, 155 sample preparation methods, 100 extraction methods, 75 instrumentation methods, 114 BGC–MS/MS links
- Intermediate per-category tallies (projects, MS samples, genomic source types, modular method fields, BGC–MS/MS link registrations)

## How to apply

Download the PoDP Zenodo deposit (regularly archived, JSON format), which contains the community-curated project metadata documents; each document conforms to the platform's JSON schema (app/public/schema.json). Parse every project JSON file with Python and aggregate: (1) number of projects, (2) MS samples with sequenced source material, (3) genomes, metagenomes, and metagenome-assembled genomes as mutually exclusive genomic-source categories, (4) distinct sample preparation methods, extraction methods, and instrumentation methods recorded in the modular metadata fields, and (5) registered BGC–MS/MS links, counting only records where a MIBiG identifier is linked with an MS/MS URL and scan number. Treat genome category assignment as the key decision point: classify each deposited genomic source into exactly one of genome, metagenome, or MAG before tallying, so category totals sum to the >2,600 distinct genomic sources. Validate by writing the computed counts to statistics_table.csv and confirming each matches the published Box 1 values — 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs, 155 sample preparation methods, 100 extraction methods, 75 instrumentation methods, and 114 BGC–MS/MS links.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project metadata (pairs of omics datasets with modular metadata fields) whose JSON export is counted) — https://github.com/iomega/paired-data-form
- **Zenodo** (Archive from which the PoDP project JSON files are downloaded for the reproducibility computation (doi:10.5281/zenodo.6091663))
- **Python** (Parser/aggregator for the JSON project documents used to compute all Box 1 counts)

## Examples

```
python -c "import json,glob; projects=[json.load(open(f)) for f in glob.glob('podp_zenodo/*.json')]; print(len(projects))"
```

## Evaluation signals

- Each computed count matches the published Box 1 value exactly: 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs, 155 sample preparation methods, 100 extraction methods, 75 instrumentation methods, 114 BGC–MS/MS links
- Genomic-source categories are mutually exclusive and sum to the 'more than 2,600 different genomic sources deposited' (1,306 + 1,268 + 42 = 2,616)
- BGC–MS/MS link count (114) only includes MIBiG identifiers linked with an MS/MS URL and scan number — links missing any of these three components are excluded
- Method counts reflect distinct modular-field values, not per-sample occurrences (155/100/75 across >4,800 genome–metabolite links)
- Output file statistics_table.csv parses cleanly and every published Box 1 row verifies True

## Limitations

- Free text entered in the 'Other' boxes of modular metadata fields is inherently not machine-readable and can cause undercounting of method variants
- Counts are a snapshot tied to the archived Zenodo upload (scheduled uploads occur monthly), so values differ from the live platform state or later deposits
- Capturing the full range of relevant variables in standardized machine-readable format was deliberately traded off for a low-burden modular entry process, so some metadata diversity is collapsed or user-customized

## Evidence

- [abstract] Genomic source category counts: "Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes."
- [abstract] Zenodo JSON archive: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Modular method metadata counts: [evidence span withheld — anchor status: not_found]
- [abstract] BGC–MS/MS link definition: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [readme] JSON schema for parsed documents: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
- [discussion] Non-machine-readable free-text caveat: "Free text entered in the “Other” boxes is inherently not machine-readable but gives an option for customization by the user"
