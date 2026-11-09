---
name: molecular-networking-genome-mining-integration
description: Use when validated BGC–metabolite links must be scored or benchmarked
  — e.g. PoDP JSON metadata with MIBiG IDs, MS/MS URLs (GNPS-MassIVE or MetaboLights),
  and genome labels are available and a candidate link ranking is needed.
when_to_use_negative:
- Your data is unpaired/unannotated omics data with no BGC–metabolite links — the
  PoDP does not store metabolomics or genomics datasets themselves, only metadata
  defining pairs, so raw unpaired inputs cannot be scored
- You need to discover BGC–MS/MS links de novo — this skill scores and ranks already
  validated or candidate links documented in PoDP metadata, it does not create the
  links
- Input metadata relies heavily on free text in 'Other' boxes, which is 'inherently
  not machine-readable' and cannot be reliably parsed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework in which the newly proposed scores are computed and candidate BGC–metabolite
    pairs are ranked
- name: Paired Omics Data Platform
  role: Source of community-curated project metadata defining genome–metabolome and
    BGC–MS/MS links, archived monthly to Zenodo as JSON
  repo: https://github.com/iomega/paired-data-form
- name: MIBiG
  role: Identifier namespace for BGCs in BGC–MS/MS links
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:ffa1fef4c44f0e39cf9117ef3f9cef4ac2c32f30f1d0faeb6218cd865929921b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - NPLinker
  - Paired Omics Data Platform
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-networking-genome-mining-integration

## Summary

Score and rank BGC–metabolite links by loading PoDP community-curated paired datasets (MIBiG identifiers linked to MS/MS URLs with scan numbers and genome labels) into the NPLinker framework and computing the newly proposed scoring system on validated links.

## When to use

Use when validated BGC–metabolite links must be scored or benchmarked — e.g. PoDP JSON metadata with MIBiG IDs, MS/MS URLs (GNPS-MassIVE or MetaboLights), and genome labels are available and a candidate link ranking is needed.

## When NOT to use

- Your data is unpaired/unannotated omics data with no BGC–metabolite links — the PoDP does not store metabolomics or genomics datasets themselves, only metadata defining pairs, so raw unpaired inputs cannot be scored
- You need to discover BGC–MS/MS links de novo — this skill scores and ranks already validated or candidate links documented in PoDP metadata, it does not create the links
- Input metadata relies heavily on free text in 'Other' boxes, which is 'inherently not machine-readable' and cannot be reliably parsed

## Inputs

- PoDP project metadata JSON files from the Zenodo archive or pairedomicsdata.bioinformatics.nl
- MIBiG identifiers for BGCs (including five BGCs newly uploaded to MIBiG per the article)
- MS/MS dataset accessions (GNPS-MassIVE or MetaboLights study identifiers) with scan numbers
- Genome public database identifiers (NCBI GenBank accession numbers or JGI Genome IDs) with user-created genome labels

## Outputs

- bgc_metabolite_scores.tsv — ranked table of validated BGC–metabolite pairs with computed NPLinker scores
- Assembled paired datasets loaded in the NPLinker framework

## How to apply

Download the community-curated PoDP project metadata as JSON from the Zenodo archive (or via the per-project download button on pairedomicsdata.bioinformatics.nl). Parse the JSON to extract validated BGC–metabolite links: MIBiG identifiers, MS/MS URLs with scan numbers, and user-created genome labels. Resolve the referenced public records — GNPS-MassIVE or MetaboLights accessions for MS/MS data, and NCBI GenBank accessions or JGI Genome IDs for genomes — which the PoDP metadata points to but does not itself store. Load the assembled datasets into NPLinker and compute the newly proposed scores for each candidate BGC–metabolite pair, then rank the validated pairs by score and write bgc_metabolite_scores.tsv. The rationale is that the PoDP was built precisely so that 'a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number', enabling systematic scoring demonstrations within NPLinker.

## Related tools

- **NPLinker** (Framework in which the newly proposed scores are computed and candidate BGC–metabolite pairs are ranked)
- **Paired Omics Data Platform** (Source of community-curated project metadata defining genome–metabolome and BGC–MS/MS links, archived monthly to Zenodo as JSON) — https://github.com/iomega/paired-data-form
- **MIBiG** (Identifier namespace for BGCs in BGC–MS/MS links)

## Evaluation signals

- Every validated BGC–metabolite link from the demonstration PoDP datasets receives a score in bgc_metabolite_scores.tsv
- Each scored pair resolves to a MIBiG identifier, an MS/MS URL with scan number, and a genome label traceable to the source PoDP project JSON
- Referenced public records resolve: MassIVE or MetaboLights accessions for MS/MS data and NCBI GenBank or JGI Genome IDs for genomes
- Ranking is monotonic with the computed scores (pairs sorted by score in the output TSV)
- Output TSV reproduces the reported demonstration of the novel scoring system within NPLinker

## Limitations

- The PoDP does not store metabolomics or genomics datasets, so all public records must be resolved externally via accession numbers
- Free text entered in 'Other' boxes is inherently not machine-readable
- Paired data generation has substantial manual burden, complicating scaling: 'manual burden of paired data creation is likely too significant for high-throughput needs'

## Evidence

- [abstract] a new set of scores was recently proposed using a number of PoDP datasets with validated BGC–metabolite links to demonstrate the effect of the novel scoring system within the newly introduced: "a new set of scores was recently proposed using a number of PoDP datasets with validated BGC–metabolite links to demonstrate the effect of the novel scoring system within the newly introduced"
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [readme] The web service has an OpenAPI (v3.0.3) specification (https://www.openapis.org/) which can be used to submit and retrieve projects in a programmatic manner.: [evidence span withheld — anchor status: not_found]
- [discussion] manual burden of paired data creation is likely too significant for high-throughput needs: [evidence span withheld — anchor status: not_found]
