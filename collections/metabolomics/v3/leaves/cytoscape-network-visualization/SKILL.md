---
name: cytoscape-network-visualization
description: Use when LC-MS/MS spectra (e.g. from a MassIVE deposit like MSV000083738)
  must be clustered into molecular families whose composition — known vs. putatively
  novel analogues — must be enumerated in a graph layout.
when_to_use_negative:
- The goal is clustering biosynthetic gene clusters rather than MS/MS spectra — that
  is BiG-SCAPE/CORASON territory, with its own distance cutoff of 0.3 and Affinity
  Propagation clustering
- The input is already a node/edge table or feature table and no spectral cosine scoring
  is needed
- Only MS1 features without fragmentation (MS/MS) scans are available, since edges
  depend on fragmentation cosine similarity
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Cytoscape
  role: Import of molecular-network edge/node tables, generation of network layouts
    where molecular families appear as connected components, and manual identification
    of the detoxin/rimosamide family
- name: BiG-SCAPE
  role: Companion network-construction framework in the same workflow; its sequence-similarity
    network paradigm (filtering by a cutoff) parallels the cosine-threshold filtering
    of the MS/MS molecular network
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: CORASON
  role: Used in the same metabologenomic pipeline to connect gene cluster families
    to molecular families; not itself the Cytoscape step but part of the integrated
    workflow this visualization supports
  repo: https://github.com/nselem/corason
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:1ea12f2018f94bdfd4caea7d080b1599009c110f87356bdab53eb07ae03b7dfa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - CORASON
  - Cytoscape
  - GNPS
  - MSCluster
  - MassIVE
  - MetaboDirect
  - pandas
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- network-visualization-in-cytoscape
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cytoscape-network-visualization

## Summary

Building and inspecting molecular networks in Cytoscape: MS/MS spectra from the 363-strain actinomycete LC-MS/MS metabolomics dataset are scored by pairwise cosine similarity, edges are filtered at cosine ≥ 0.6, and connected components (molecular families) are visually analyzed to assign known detoxin/rimosamide ions and count novel analogues.

## When to use

Use when LC-MS/MS spectra (e.g. from a MassIVE deposit like MSV000083738) must be clustered into molecular families whose composition — known vs. putatively novel analogues — must be enumerated in a graph layout.

## When NOT to use

- The goal is clustering biosynthetic gene clusters rather than MS/MS spectra — that is BiG-SCAPE/CORASON territory, with its own distance cutoff of 0.3 and Affinity Propagation clustering
- The input is already a node/edge table or feature table and no spectral cosine scoring is needed
- Only MS1 features without fragmentation (MS/MS) scans are available, since edges depend on fragmentation cosine similarity

## Inputs

- LC-MS/MS data files from the 363-strain actinomycete metabolomics study (MassIVE accession MSV000083738)
- MS/MS peak lists preprocessed by removing the 25% lowest-intensity ions per scan
- Reference MIBiG entries BGC0001840, BGC0001878, BGC0001841 for known detoxins and rimosamides

## Outputs

- Molecular network in GraphML format for import into Cytoscape
- Cytoscape network layout in which molecular families appear as connected components
- detoxin_rimosamide_family_report.csv with the family breakdown (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues)

## How to apply

Download the deposited LC-MS/MS files (MassIVE MSV000083738 for the 363-strain study) and preprocess each MS/MS scan by removing the 25% of ions with the lowest intensity, as in the study's metabolomic pipeline. Compute pairwise cosine-similarity scores between MS/MS spectra and retain only edges with cosine similarity ≥ 0.6 when building the molecular network (note the metabologenomic ion-identity step used a stricter cosine > 0.75 with mass within 4 ppm). Import the resulting edge and node tables into Cytoscape and generate a layout in which molecular families appear as connected components. Manually analyze the network to locate the detoxin/rimosamide molecular family, referencing MIBiG accessions BGC0001840, BGC0001878 and BGC0001841 for the known compounds. Validate by confirming the family composition of 110 molecules (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues).

## Related tools

- **Cytoscape** (Import of molecular-network edge/node tables, generation of network layouts where molecular families appear as connected components, and manual identification of the detoxin/rimosamide family)
- **BiG-SCAPE** (Companion network-construction framework in the same workflow; its sequence-similarity network paradigm (filtering by a cutoff) parallels the cosine-threshold filtering of the MS/MS molecular network) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **CORASON** (Used in the same metabologenomic pipeline to connect gene cluster families to molecular families; not itself the Cytoscape step but part of the integrated workflow this visualization supports) — https://github.com/nselem/corason

## Evaluation signals

- The exported molecular network exists in GraphML format and opens in Cytoscape with molecular families visible as connected components
- The family report reproduces the 110-member detoxin/rimosamide family with the 8/4/99 composition breakdown (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues)
- Edge list contains only pairwise spectrum links with cosine similarity ≥ 0.6
- Preprocessing removed the 25% of ions with the lowest intensity in each MS/MS scan before scoring
- Identified known ions map to compounds referenced under MIBiG accessions BGC0001840, BGC0001878 and BGC0001841

## Limitations

- Molecular network family identification was performed manually — the network was manually analyzed to identify ions related to known detoxins and rimosamides, so family composition depends on the analyst
- LC-MS acquisition parameters were occasionally adjusted rather than fully standardized, which can affect spectral comparability
- MS/MS data for non-detoxin-producing strains was not deposited and is available only upon request, limiting reproducibility of the full network

## Evidence
