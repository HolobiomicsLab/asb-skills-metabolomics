---
name: bgc-msms-linkage-analysis
description: Use when candidate BGC–metabolite (BGC–MS/MS) links need to be scored
  and ranked within NPLinker, and validated paired-omics links are available from
  the PoDP as JSON with MIBiG identifiers and MS/MS URLs with scan numbers.
when_to_use_negative:
- When the metabolomics or genomics datasets are not deposited in public repositories
  identifiable by MassIVE/MetaboLights and GenBank/JGI identifiers, since the PoDP
  captures only metadata defining pairs of omics datasets in existing public databases
  and stores none of the raw data.
- When the analysis goal is sample-level genome–metabolome linking without BGC-level
  resolution — the PoDP BGC–MS/MS link requires a MIBiG identifier, which unannotated
  candidate BGCs lack.
- When free-text metadata entered in 'Other' boxes is the primary analysis variable,
  as it is inherently not machine-readable.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: NPLinker
  role: Framework in which the newly proposed scores are computed for each candidate
    BGC–metabolite pair and the validated links are ranked
- name: Paired Omics Data Platform
  role: Source of community-curated project metadata defining validated BGC–metabolite
    links, archived to Zenodo as JSON
  repo: https://github.com/iomega/paired-data-form
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:bf7d47b3a25add02880f3aec81fef7fab71d2a30076cc2bc0defd6e9076eef7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - NPLinker
  - Paired Omics Data Platform
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bgc-msms-linkage-analysis

## Summary

Computes and ranks scores for candidate BGC–metabolite pairs by loading community-curated Paired Omics Data Platform (PoDP) validated links into the NPLinker framework, demonstrating a novel scoring system on systematically paired genomic and metabolomic datasets.

## When to use

Use when candidate BGC–metabolite (BGC–MS/MS) links need to be scored and ranked within NPLinker, and validated paired-omics links are available from the PoDP as JSON with MIBiG identifiers and MS/MS URLs with scan numbers.

## When NOT to use

- When the metabolomics or genomics datasets are not deposited in public repositories identifiable by MassIVE/MetaboLights and GenBank/JGI identifiers, since the PoDP captures only metadata defining pairs of omics datasets in existing public databases and stores none of the raw data.
- When the analysis goal is sample-level genome–metabolome linking without BGC-level resolution — the PoDP BGC–MS/MS link requires a MIBiG identifier, which unannotated candidate BGCs lack.
- When free-text metadata entered in 'Other' boxes is the primary analysis variable, as it is inherently not machine-readable.

## Inputs

- PoDP project metadata in JSON format (from the Zenodo archive or pairedomicsdata.bioinformatics.nl per-project download)
- MIBiG identifiers for BGCs (same or similar BGC)
- MS/MS URLs with scan numbers resolving to GNPS-MassIVE or MetaboLights submissions
- Genome labels tied to NCBI GenBank accession numbers or JGI Genome IDs
- Optional BioSample accession IDs linking external metadata to (meta)genomes

## Outputs

- bgc_metabolite_scores.tsv — ranked table of validated BGC–metabolite pairs with computed NPLinker scores
- Resolved public dataset records (GenBank/JGI genome entries and GNPS-MassIVE/MetaboLights MS/MS submissions) loaded into the NPLinker framework

## How to apply

Download the community-curated PoDP project metadata in JSON format from the Zenodo archive (or per-project download from pairedomicsdata.bioinformatics.nl); each project is organized as multiple related experiments identified by MassIVE accession or MetaboLights study identifier, with (meta)genomes added via a public database identifier (NCBI GenBank accession or JGI Genome ID) and user-created genome labels. Parse the JSON to extract validated BGC–metabolite links: a MIBiG identifier for the same or similar BGC linked with an MS/MS URL and scan number. Resolve the referenced public records — GNPS-MassIVE or MetaboLights accessions for MS/MS data and GenBank/JGI identifiers for genomes — then load the assembled datasets into NPLinker and compute the newly proposed scores for each candidate BGC–metabolite pair. Rank the validated pairs by score and write the result to bgc_metabolite_scores.tsv. The rationale is that PoDP's systematically curated, community-validated links provide ground truth against which the effect of the novel scoring system within NPLinker can be demonstrated.

## Related tools

- **NPLinker** (Framework in which the newly proposed scores are computed for each candidate BGC–metabolite pair and the validated links are ranked)
- **Paired Omics Data Platform** (Source of community-curated project metadata defining validated BGC–metabolite links, archived to Zenodo as JSON) — https://github.com/iomega/paired-data-form

## Evaluation signals

- Every validated BGC–metabolite link from the demonstration PoDP datasets receives a score in bgc_metabolite_scores.tsv.
- Each scored pair carries a resolvable MIBiG identifier and an MS/MS URL with scan number pointing to a GNPS-MassIVE or MetaboLights record.
- Each genome label resolves to a public database identifier (NCBI GenBank accession or JGI Genome ID) in the project metadata.
- Re-running the scoring on the same Zenodo-archived PoDP JSON reproduces the same ranking of pairs, demonstrating the reported effect of the novel scoring system within NPLinker.

## Limitations

- Free text entered in 'Other' boxes in PoDP metadata is inherently not machine-readable, limiting automated parsing of some project fields.
- The manual burden of paired data creation is likely too significant for high-throughput needs, so validated link coverage may lag behind available unpaired datasets.
- MS2 matches may be non-identical due to impure isolations, making MS2 spectral matching non-deterministic for some BGC–metabolite links.
- Some structural questions cannot be resolved by the links alone — e.g., esters not unambiguously identifiable based on LC-MS/MS fragmentation patterns, requiring spectra of additional reference compounds for discrimination.

## Evidence

- [abstract] Novel scoring demonstrated on validated links: "a new set of scores was recently proposed using a number of PoDP datasets with validated BGC–metabolite links to demonstrate the effect of the novel scoring system within the newly introduced"
- [abstract] BGC–MS/MS link construction: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] Zenodo JSON archive of curated data: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] Project metadata organization: "The metadata is organized in projects that can consist of multiple related experiments, identified by their MassIVE accession or MetaboLights study identifier"
- [readme] Platform resolution of public identifiers: [evidence span withheld — anchor status: external_doc]
