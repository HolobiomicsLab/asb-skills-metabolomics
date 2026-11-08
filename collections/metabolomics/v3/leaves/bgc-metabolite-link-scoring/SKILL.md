---
name: bgc-metabolite-link-scoring
description: Use when you have PoDP project metadata (JSON from Zenodo or pairedomicsdata.bioinformatics.nl)
  with validated BGC–metabolite links and need to compute and rank scores for candidate
  BGC–metabolite pairs in NPLinker.
when_to_use_negative:
- Input omics data are raw MS/MS or genome files without accompanying PoDP project
  metadata defining the pairs — the skill depends on curated MIBiG/MS/MS URL/genome-label
  links
- The goal is genome–metabolome (sample-level) pairing or PoDP project creation/submission
  rather than scoring BGC–metabolite links in NPLinker
- Free-text metadata entered in 'Other' boxes is the only link information available,
  since it is inherently not machine-readable
tools:
- name: NPLinker
  role: Framework in which the newly proposed scores are computed and ranked for candidate
    BGC–metabolite pairs from PoDP datasets
- name: Paired Omics Data Platform
  role: Source of community-curated project metadata (JSON schema, OpenAPI v3.0.3
    service, monthly Zenodo archive) that provides the validated BGC–metabolite links
    to be scored
  repo: https://github.com/iomega/paired-data-form
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:c65e4583182d989342734de6056865bbbbd43ec8f9eb1340781249959a492665
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - NPLinker
  - Paired Omics Data Platform
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bgc-metabolite-link-scoring

## Summary

Scores and ranks BGC–metabolite links by loading community-curated PoDP validated links (MIBiG identifiers paired with MS/MS URLs and genome labels) into the NPLinker framework and computing the newly proposed scoring system on the resolved public datasets.

## When to use

Use when you have PoDP project metadata (JSON from Zenodo or pairedomicsdata.bioinformatics.nl) with validated BGC–metabolite links and need to compute and rank scores for candidate BGC–metabolite pairs in NPLinker.

## When NOT to use

- Input omics data are raw MS/MS or genome files without accompanying PoDP project metadata defining the pairs — the skill depends on curated MIBiG/MS/MS URL/genome-label links
- The goal is genome–metabolome (sample-level) pairing or PoDP project creation/submission rather than scoring BGC–metabolite links in NPLinker
- Free-text metadata entered in 'Other' boxes is the only link information available, since it is inherently not machine-readable

## Inputs

- PoDP project metadata in JSON format (Zenodo archive or pairedomicsdata.bioinformatics.nl download)
- Validated BGC–metabolite links: MIBiG identifiers, MS/MS URLs with scan numbers, genome labels
- GNPS-MassIVE or MetaboLights accessions for MS/MS datasets
- NCBI GenBank accessions or JGI Genome IDs for (meta)genomes

## Outputs

- bgc_metabolite_scores.tsv — ranked table of validated BGC–metabolite pairs with their computed scores
- Per-pair score assignments for every validated BGC–metabolite link in the demonstration datasets

## How to apply

Download the community-curated PoDP project metadata in JSON format from the Zenodo archive (doi 10.5281/zenodo.6091663) or via the per-project download button, and parse it to extract validated BGC–metabolite links, capturing the MIBiG identifier, the MS/MS URL with scan number, and the user-created genome label for each pair. Resolve the underlying public records referenced by each project: GNPS-MassIVE or MetaboLights accessions for the MS/MS data, and NCBI GenBank accessions or JGI Genome IDs for the (meta)genomes; species names can be fetched from GenBank using the public genome identifiers. Load the assembled paired datasets into the NPLinker framework and compute the newly proposed scores for each candidate BGC–metabolite pair. Rank the validated pairs by score and write the ranked result to bgc_metabolite_scores.tsv. The rationale is that the PoDP projects (70 paired datasets, >4,800 genome–metabolite links and 114 BGC–MS/MS links) provide validated ground-truth links that make scoring schemes benchmarkable within NPLinker.

## Related tools

- **NPLinker** (Framework in which the newly proposed scores are computed and ranked for candidate BGC–metabolite pairs from PoDP datasets)
- **Paired Omics Data Platform** (Source of community-curated project metadata (JSON schema, OpenAPI v3.0.3 service, monthly Zenodo archive) that provides the validated BGC–metabolite links to be scored) — https://github.com/iomega/paired-data-form

## Evaluation signals

- Every validated BGC–metabolite link from the demonstration datasets receives a score in the output table, reproducing the reported demonstration of the novel scoring system within NPLinker
- bgc_metabolite_scores.tsv exists and rows are ranked by score
- Each scored link can be traced back to a MIBiG identifier, an MS/MS URL with scan number, and a genome label from the parsed PoDP JSON
- All referenced GNPS-MassIVE/MetaboLights and GenBank/JGI public identifiers resolve without error
- Scores recomputed from the same Zenodo-archived JSON snapshot are reproducible

## Limitations

- Free text entered in the PoDP 'Other' boxes is inherently not machine-readable, so customized metadata cannot be reliably parsed for scoring
- Paired data creation carries a substantial manual burden, limiting how much validated-link data can be scaled up
- MS2 spectral matches may be non-identical due to impure isolations, making spectral matching of linked metabolites non-deterministic

## Evidence

- [abstract] a new set of scores was recently proposed using a number of PoDP datasets with validated BGC–metabolite links to demonstrate the effect of the novel scoring system within the newly introduced: "a new set of scores was recently proposed using a number of PoDP datasets with validated BGC–metabolite links to demonstrate the effect of the novel scoring system within the newly introduced"
- [abstract] Zenodo JSON archive: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [abstract] BGC–MS/MS link structure: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [readme] Programmatic project retrieval: [evidence span withheld — anchor status: not_found]
