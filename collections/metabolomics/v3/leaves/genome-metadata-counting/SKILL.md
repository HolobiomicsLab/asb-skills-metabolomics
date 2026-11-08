---
name: genome-metadata-counting
description: Use when the task requires reproducing or auditing aggregate counts (projects,
  MS samples, genomes/metagenomes/MAGs, method counts, BGC–MS/MS links) from PoDP
  Zenodo JSON archives, or verifying reported PoDP corpus statistics.
when_to_use_negative:
- The input is raw metabolomics spectra (GNPS-MassIVE or MetaboLights submissions)
  rather than PoDP project JSON documents — this skill only aggregates PoDP metadata,
  it does not process spectra
- The question concerns genome–metabolome pairing workflow steps (depositing data,
  linking genome labels with MS spectra URLs) rather than reproducing corpus-level
  statistics
- The data is free-text entered in 'Other' metadata boxes, which the paper notes is
  inherently not machine-readable and cannot be reliably counted
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project metadata whose JSON archives are counted;
    projects record (meta)genomes via public database identifiers and modular sample
    preparation/extraction/instrumentation fields
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Archive where the PoDP JSON project files are deposited and from which the
    statistics are reproduced
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:dd8c0381de2c3006a35ac9c7165f1689abd5ec8a40116e083c38697ac29395a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# genome-metadata-counting

## Summary

Reproduces preliminary dataset statistics for the Paired Omics Data Platform (PoDP) by parsing the community-curated project JSON documents archived to Zenodo and aggregating counts of genomes, metagenomes, metagenome-assembled genomes, MS samples, metadata methods, and BGC–MS/MS links against the paper's Box 1 values.

## When to use

Use when the task requires reproducing or auditing aggregate counts (projects, MS samples, genomes/metagenomes/MAGs, method counts, BGC–MS/MS links) from PoDP Zenodo JSON archives, or verifying reported PoDP corpus statistics.

## When NOT to use

- The input is raw metabolomics spectra (GNPS-MassIVE or MetaboLights submissions) rather than PoDP project JSON documents — this skill only aggregates PoDP metadata, it does not process spectra
- The question concerns genome–metabolome pairing workflow steps (depositing data, linking genome labels with MS spectra URLs) rather than reproducing corpus-level statistics
- The data is free-text entered in 'Other' metadata boxes, which the paper notes is inherently not machine-readable and cannot be reliably counted

## Inputs

- PoDP Zenodo deposit (community-curated project metadata as JSON documents)
- Paired-data-form project JSON schema (app/public/schema.json)
- Published Box 1 preliminary dataset statistics (targets for validation)

## Outputs

- statistics_table.csv with computed counts reconciled against Box 1
- Per-category counts: genomes (1,268), metagenomes (1,306), MAGs (42), MS samples (4,853), methods (155/100/75), BGC–MS/MS links (114)

## How to apply

Download the PoDP Zenodo deposit, which contains the community-curated project metadata archived monthly as JSON documents conforming to the paired-data-form JSON schema (app/public/schema.json). Parse all project JSON files with Python and, for each project, classify the (meta)genomic sources via their public database identifiers (NCBI GenBank accession or JGI Genome ID) into genomes, metagenomes, and metagenome-assembled genomes; count MS samples with sequenced source material. Aggregate counts of distinct user-recorded modular metadata fields: sample preparation methods, extraction methods, and instrumentation methods. Count BGC–MS/MS links only where a MIBiG identifier is linked with both an MS/MS URL and a scan number, since this is how the paper defines a link. Validation requires each computed value to match Box 1 exactly: 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs, 155 sample preparation methods, 100 extraction methods, 75 instrumentation methods, and 114 BGC–MS/MS links; write the reconciliation to statistics_table.csv.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project metadata whose JSON archives are counted; projects record (meta)genomes via public database identifiers and modular sample preparation/extraction/instrumentation fields) — https://github.com/iomega/paired-data-form
- **Zenodo** (Archive where the PoDP JSON project files are deposited and from which the statistics are reproduced)

## Examples

```
python -c "import json,glob; projs=[json.load(open(f)) for f in glob.glob('podp_*.json')]; from collections import Counter; print(Counter(s['type'] for p in projs for s in p.get('genomes',[])))"
```

## Evaluation signals

- Computed genome/metagenome/MAG counts match the abstract values: 1,306 metagenomes, 1,268 genomes, and 42 metagenome-assembled genomes among more than 2,600 genomic sources
- Computed total genomic sources exceed 2,600, consistent with the sum of the three source categories
- Project count equals 70 and MS-sample-with-sequenced-source-material count equals 4,853 as reported in Box 1
- Modular metadata counts equal 155 sample preparation methods, 100 extraction methods, and 75 instrumentation methods
- BGC–MS/MS link count equals 114, where links are defined as MIBiG identifiers paired with MS/MS URLs and scan numbers

## Limitations

- Free text entered in 'Other' boxes is inherently not machine-readable but gives an option for customization by the user, so such entries cannot be systematically counted
- Capturing the full range of relevant variables in a standardized machine-readable format is complex and tedious, so the metadata (and hence counts) reflect minimal, modular user-entered records
- The PoDP stores only metadata, not the metabolomics or genomics datasets themselves; counts depend on the Zenodo archive version and change as the community submits new paired data

## Evidence

- [abstract] Counts of genomic source types: "Of the more than 2,600 different genomic sources deposited, 1,306 are metagenomes, 1,268 are genomes, and 42 are metagenome-assembled genomes."
- [abstract] Box 1 targets and metadata counts: "Over 45 laboratories from 10 countries have contributed 70 paired datasets. Those 70 projects (Box 1) contain 4,853 MS samples associated with sequenced source material."
- [abstract] Modular metadata field counts: [evidence span withheld — anchor status: not_found]
- [abstract] BGC–MS/MS link definition: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] JSON format and Zenodo archiving of PoDP metadata: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] JSON schema governing the counted project files: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
