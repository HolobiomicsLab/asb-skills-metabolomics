---
name: metabolome-genome-link-tabulation
description: Use when you have downloaded the PoDP Zenodo JSON deposit and need to
  reproduce or audit the published dataset statistics (Box 1) — e.g. validating that
  1,306 metagenomes, 1,268 genomes, and 42 MAGs are present.
when_to_use_negative:
- You need the raw metabolomics or genomics data itself — the PoDP captures only metadata
  defining pairs of omics datasets in existing public databases and stores no spectra
  or sequences
- You want to enumerate individual genome–metabolite link instances rather than reproduce
  aggregate corpus-level statistics
- The metadata you need lives in free-text 'Other' boxes, which is inherently not
  machine-readable and cannot be tabulated programmatically
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project metadata whose JSON files are parsed
    and aggregated; projects link MS/MS spectra with genome, sample preparation, extraction,
    and instrumentation metadata
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Archive from which the PoDP JSON project files are downloaded on a monthly
    scheduled upload
- name: MIBiG
  role: Identifier namespace used to count BGC–MS/MS links (a MIBiG identifier linked
    with a MS/MS URL and scan number)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:afc401f6374b02752b7e7ecd9e5926f3a0d911c0c4dd1b33ab2be5fba008d735
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolome-genome-link-tabulation

## Summary

Reproduce PoDP preliminary dataset statistics by parsing the community-curated JSON project metadata archived to Zenodo and aggregating counts of projects, MS samples, genomes, metagenomes, MAGs, metadata methods, and registered BGC–MS/MS links.

## When to use

Use when you have downloaded the PoDP Zenodo JSON deposit and need to reproduce or audit the published dataset statistics (Box 1) — e.g. validating that 1,306 metagenomes, 1,268 genomes, and 42 MAGs are present.

## When NOT to use

- You need the raw metabolomics or genomics data itself — the PoDP captures only metadata defining pairs of omics datasets in existing public databases and stores no spectra or sequences
- You want to enumerate individual genome–metabolite link instances rather than reproduce aggregate corpus-level statistics
- The metadata you need lives in free-text 'Other' boxes, which is inherently not machine-readable and cannot be tabulated programmatically

## Inputs

- PoDP Zenodo deposit of community-curated project metadata as JSON documents (paired-data-form schema)
- Published Box 1 preliminary dataset statistics used as validation targets

## Outputs

- statistics_table.csv with aggregated counts of projects, MS samples, genomes, metagenomes, MAGs, preparation/extraction/instrumentation methods, and BGC–MS/MS links
- Per-category counts matching or deviating from published values (70 projects, 4,853 MS samples, 1,306 metagenomes, 1,268 genomes, 42 MAGs, 155/100/75 methods, 114 BGC–MS/MS links)

## How to apply

Download the PoDP Zenodo deposit, which contains the community-curated project metadata as JSON documents conforming to the paired-data-form schema. Parse all JSON project files with Python and aggregate counts of projects, MS samples with sequenced source material, genomes, metagenomes, and metagenome-assembled genomes. Separately aggregate counts of distinct sample preparation methods, extraction methods, and instrumentation methods recorded in the modular metadata fields, then count all registered BGC–MS/MS links, defined as MIBiG identifiers linked with MS/MS URLs and scan numbers. Validate each computed count against the published Box 1 values — 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs, 155 sample preparation methods, 100 extraction methods, 75 instrumentation methods, and 114 BGC–MS/MS links — and write the result as statistics_table.csv.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project metadata whose JSON files are parsed and aggregated; projects link MS/MS spectra with genome, sample preparation, extraction, and instrumentation metadata) — https://github.com/iomega/paired-data-form
- **Zenodo** (Archive from which the PoDP JSON project files are downloaded on a monthly scheduled upload)
- **MIBiG** (Identifier namespace used to count BGC–MS/MS links (a MIBiG identifier linked with a MS/MS URL and scan number))

## Examples

```
python -c "import json,glob,collections; c=collections.Counter(); [c.update([1]) for f in glob.glob('podp/*.json') for p in [json.load(open(f))] ] ; print(c)"  # aggregate project JSON fields into statistics_table.csv
```

## Evaluation signals

- Computed metagenome count equals 1,306 and genome count equals 1,268 among the more than 2,600 genomic sources
- Metagenome-assembled genome count equals 42
- Project count equals 70 and MS samples with sequenced source material equals 4,853
- Distinct method counts equal 155 sample preparation methods, 100 extraction methods, and 75 instrumentation methods
- Registered BGC–MS/MS links (MIBiG identifier + MS/MS URL + scan number) equal 114; output written as statistics_table.csv

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so customized metadata cannot be captured in the aggregation
- Counts reflect a snapshot; the platform uploads projects to Zenodo each month, so the deposit grows over time and published Box 1 values may lag the current archive
- Capturing the full range of relevant experimental variables in standardized machine-readable format is incomplete, which bounds what can be tabulated

## Evidence

- [abstract] Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes.: "Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes."
- [abstract] The impressive collection of over 4,800 genome–metabolite links is accompanied by metadata: 155 sample preparation methods, 100 extraction methods, and 75 instrumentation methods.: [evidence span withheld — anchor status: not_found]
- [abstract] To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The web service uses a redis queue (v5.0.5) to schedule jobs to fetch more information about the public identifiers and to upload the projects to Zenodo each month.: [evidence span withheld — anchor status: external_doc]
