---
name: msms-molecular-networking
description: Use when raw LC-MS/MS metabolomics files (e.g. mzML/MassIVE MSV000083738)
  must be grouped into molecular families to identify analogues of known metabolites
  such as detoxins and rimosamides.
when_to_use_negative:
- The goal is to cluster biosynthetic gene clusters from genomes — use BiG-SCAPE/CORASON
  sequence similarity networking instead of MS/MS spectral networking
- MS/MS data for the strains of interest is not deposited (in this study, MS/MS data
  for non-detoxin-relevant strains is available only upon request)
- A fully automated, non-manual molecular family annotation is required — the original
  study identified the detoxin/rimosamide family by manual network analysis
tools:
- name: Cytoscape
  role: Visualization of the molecular network; connected components are inspected
    as molecular families
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:84b05947b66b4eb11d470e0d20df912e3d893f1043802933169daf615b577c16
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cytoscape
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# msms-molecular-networking

## Summary

Builds molecular families from LC-MS/MS metabolomics data by cosine-similarity networking of MS/MS spectra, as used to map the detoxin/rimosamide molecular family across the 363-strain actinomycete metabolomic dataset (MassIVE MSV000083738).

## When to use

Use when raw LC-MS/MS metabolomics files (e.g. mzML/MassIVE MSV000083738) must be grouped into molecular families to identify analogues of known metabolites such as detoxins and rimosamides.

## When NOT to use

- The goal is to cluster biosynthetic gene clusters from genomes — use BiG-SCAPE/CORASON sequence similarity networking instead of MS/MS spectral networking
- MS/MS data for the strains of interest is not deposited (in this study, MS/MS data for non-detoxin-relevant strains is available only upon request)
- A fully automated, non-manual molecular family annotation is required — the original study identified the detoxin/rimosamide family by manual network analysis

## Inputs

- LC-MS/MS metabolomics data files (MassIVE accession MSV000083738, 363-strain actinomycete study)
- MIBiG reference accessions BGC0001840, BGC0001878, BGC0001841 for known detoxins and rimosamides

## Outputs

- Molecular network edge/node tables and graphml export importable into Cytoscape
- detoxin_rimosamide_family_report.csv with the 110-member family breakdown (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues)

## How to apply

Download the deposited LC-MS/MS files (MassIVE MSV000083738) and preprocess MS/MS scans by removing the 25% of ions with the lowest intensity, matching the study's metabolomic pipeline. Compute pairwise cosine-similarity scores between MS/MS spectra and build the molecular network, retaining only edges with cosine similarity >= 0.6; ion identity correlations in the same study used mass tolerance within 4 ppm and cosine similarity above 0.75. Import the resulting edge/node tables into Cytoscape and generate a layout where molecular families appear as connected components. Locate the target molecular family by manually analyzing the network for ions related to known detoxins and rimosamides, referencing MIBiG accessions BGC0001840, BGC0001878 and BGC0001841. Report the family composition — the detoxin/rimosamide family contains 8 known detoxins, 4 known rimosamides and 99 putatively novel analogues (110 molecules total) — and export the network in graphml format plus a family report CSV.

## Related tools

- **Cytoscape** (Visualization of the molecular network; connected components are inspected as molecular families)

## Evaluation signals

- The exported network exists in graphml format and imports into Cytoscape
- The family report reproduces the 110-member detoxin/rimosamide family with the 8 known detoxins / 4 known rimosamides / 99 putatively novel analogues breakdown
- MS/MS scan preprocessing removed the 25% lowest-intensity ions per scan, as in the study's pipeline
- Network edges satisfy the cosine similarity >= 0.6 retention threshold
- Known detoxin/rimosamide ions are placed in the same connected component, anchored to MIBiG BGC0001840, BGC0001878, BGC0001841

## Limitations

- Molecular network family identification was performed manually ('The network was manually analyzed to identify ions related to known detoxins and rimosamides'), so results depend on expert curation
- MS/MS data for non-detoxin-producing strains was not deposited in this study and is available only upon request, limiting reproduction of the full network
- Ion identity correlations require tight mass matching (within 4 ppm) and cosine similarity above 0.75; looser parameters change which ions are retained (5,824 ions detected in two or more strains under these criteria)

## Evidence
