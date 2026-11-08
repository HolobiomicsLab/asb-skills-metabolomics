---
name: podp-json-metadata-aggregation
description: Use when you have downloaded the PoDP Zenodo JSON archive and need to
  reproduce or update aggregate counts (70 projects, 4,853 MS samples, 1,268 genomes,
  1,306 metagenomes, 42 MAGs, method counts, 114 BGC–MS/MS links).
when_to_use_negative:
- You need the actual metabolomics or genomics datasets — the PoDP does not store
  any datasets, only metadata defining pairs of omics data in public databases (MassIVE,
  MetaboLights, GenBank, JGI)
- Your input is raw instrument data (e.g. mzML) or a mass-spectra file rather than
  PoDP project JSON
- You want to create or edit paired-omics projects rather than aggregate existing
  ones — use the paired-data-form web application instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project metadata that is monthly archived
    to Zenodo as JSON and parsed here
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived PoDP JSON dataset that is downloaded and aggregated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:a5e6130f5a1aa763f30d470bbde5734f15bddd644b71986f06e54077c3305ba8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# podp-json-metadata-aggregation

## Summary

Parses the community-curated PoDP project JSON documents archived on Zenodo and aggregates them into the preliminary dataset statistics reported in Box 1 of the Paired Omics Data Platform paper (project, MS sample, genome/metagenome/MAG, method, and BGC–MS/MS link counts).

## When to use

Use when you have downloaded the PoDP Zenodo JSON archive and need to reproduce or update aggregate counts (70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs, method counts, 114 BGC–MS/MS links).

## When NOT to use

- You need the actual metabolomics or genomics datasets — the PoDP does not store any datasets, only metadata defining pairs of omics data in public databases (MassIVE, MetaboLights, GenBank, JGI)
- Your input is raw instrument data (e.g. mzML) or a mass-spectra file rather than PoDP project JSON
- You want to create or edit paired-omics projects rather than aggregate existing ones — use the paired-data-form web application instead

## Inputs

- PoDP Zenodo deposit (doi 10.5281/zenodo.6091663) of community-curated project JSON documents
- Published Box 1 preliminary dataset statistics used as validation targets

## Outputs

- statistics_table.csv with counts of projects, MS samples, genomes, metagenomes, MAGs, sample preparation methods, extraction methods, instrumentation methods, and BGC–MS/MS links
- Per-project parsed JSON records aggregated into count totals

## How to apply

Download the Zenodo deposit (doi 10.5281/zenodo.6091663), which contains the community-curated project metadata as JSON documents conforming to the paired-data-form schema (app/public/schema.json). Parse every JSON project file with Python and tally: projects; MS samples with sequenced source material; genomes vs metagenomes vs metagenome-assembled genomes; distinct sample preparation, extraction, and instrumentation methods from the modular metadata fields; and BGC–MS/MS links, defined as MIBiG identifiers linked with an MS/MS URL and scan number. Decide category membership from the genome-type fields so that genomes (1,268), metagenomes (1,306), and MAGs (42) are counted separately among the 2,600+ genomic sources. Validate by checking each computed count against the published Box 1 values, and emit the comparison as statistics_table.csv.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project metadata that is monthly archived to Zenodo as JSON and parsed here) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived PoDP JSON dataset that is downloaded and aggregated)

## Examples

```
import json, glob; projects=[json.load(open(f)) for f in glob.glob('podp/*.json')]; genomes=sum(1 for p in projects if p.get('genome_type')=='genome'); print(genomes)
```

## Evaluation signals

- Computed genome count equals 1,268, metagenome count equals 1,306, and MAG count equals 42 (sum > 2,600 genomic sources)
- Project count equals 70 and MS samples with sequenced source material equal 4,853
- Counts of distinct sample preparation methods (155), extraction methods (100), and instrumentation methods (75) match Box 1
- Registered BGC–MS/MS links (MIBiG identifier linked with MS/MS URL and scan number) total 114
- Each parsed file validates against the paired-data-form JSON schema (app/public/schema.json)

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so some modular metadata fields cannot be aggregated reliably
- The Zenodo archive is a monthly snapshot, so counts drift as new projects are deposited; the Box 1 numbers correspond to the paper's snapshot
- Capturing the full range of relevant variables in standardized machine-readable format was deliberately avoided, so aggregation is limited to the recorded fields

## Evidence

- [abstract] Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes.: "Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes."
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The web service uses a redis queue (v5.0.5) to schedule jobs to fetch more information about the public identifiers and to upload the projects to Zenodo each month.: [evidence span withheld — anchor status: external_doc]
- [abstract] To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
