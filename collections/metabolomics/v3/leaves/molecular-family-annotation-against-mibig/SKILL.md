---
name: molecular-family-annotation-against-mibig
description: Use when an LC-MS/MS molecular network contains unannotated connected
  components whose members should be tied to known natural product classes via MIBiG
  reference accessions (e.g. BGC0001840, BGC0001878, BGC0001841) and strain-level
  co-occurrence.
when_to_use_negative:
- Input is already an annotated feature table with identified compounds — no spectral
  networking or MIBiG cross-referencing is needed
- The goal is de novo compound structure elucidation; this skill only relates mass
  features to known MIBiG reference BGCs and flags putative analogues
- Only a single strain or no strain-level replicate data is available, since correlatable
  GCFs required representatives in two or more strains
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: Cytoscape
  role: Visualization and layout of the molecular network so molecular families appear
    as connected components for manual annotation against MIBiG references
- name: BiG-SCAPE
  role: Generates BGC sequence similarity networks and GCFs from antiSMASH/MIBiG inputs
    that are matched to molecular families via metabologenomic correlation
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: CORASON
  role: Resolves evolutionary relationships within GCFs linked to the molecular family,
    supporting prioritization of the detoxin/rimosamide BGCs
  repo: https://github.com/nselem/corason
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:c2dc683233d47616b838f2eee630d73338b554583ce07200626540dd329609a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - CORASON
  - Cytoscape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# molecular-family-annotation-against-mibig

## Summary

Annotates molecular families from LC-MS/MS molecular networking by linking uncharacterized mass features to known biosynthetic gene clusters via MIBiG reference compounds and metabologenomic correlation, as demonstrated for the detoxin/rimosamide family in a 363-strain actinomycete study.

## When to use

Use when an LC-MS/MS molecular network contains unannotated connected components whose members should be tied to known natural product classes via MIBiG reference accessions (e.g. BGC0001840, BGC0001878, BGC0001841) and strain-level co-occurrence.

## When NOT to use

- Input is already an annotated feature table with identified compounds — no spectral networking or MIBiG cross-referencing is needed
- The goal is de novo compound structure elucidation; this skill only relates mass features to known MIBiG reference BGCs and flags putative analogues
- Only a single strain or no strain-level replicate data is available, since correlatable GCFs required representatives in two or more strains

## Inputs

- LC-MS/MS metabolomics data files (MassIVE accession MSV000083738) from the 363-strain actinomycete study
- Molecular network edge and node tables (cosine-similarity graph) for import into Cytoscape
- MIBiG reference BGC accessions for known compounds (e.g. BGC0001840, BGC0001878, BGC0001841)
- antiSMASH-predicted BGCs / BiG-SCAPE GCF assignments for the same strains (for metabologenomic correlation)

## Outputs

- Cytoscape-compatible molecular network in graphml format with molecular families as connected components
- detoxin_rimosamide_family_report.csv reproducing the 110-member family composition (8 known detoxins / 4 known rimosamides / 99 putatively novel analogues)
- Annotated molecular family membership mapping mass features to known reference compounds and putative novel analogues

## How to apply

Preprocess MS/MS scans by removing the 25% of ions with the lowest intensity, then compute pairwise cosine similarities to build the molecular network; the study's metabologenomic pipeline required fragment mass within 4 ppm and cosine similarity above 0.75 to call ion identities across strains, and the detoxin/rimosamide task retained network edges at cosine similarity >= 0.6. Import the resulting edge/node tables into Cytoscape so molecular families appear as connected components, and cross-reference known compounds by their MIBiG accessions (BGC0001840, BGC0001878, BGC0001841 for detoxins/rimosamides). Link molecular families (MFs) to gene cluster families (GCFs) predicted from the same strains with BiG-SCAPE, considering only GCFs with representatives in two or more strains as correlatable. Identify the target family by manually analyzing the network for ions related to the known reference compounds, then verify composition — here 110 members: 8 known detoxins, 4 known rimosamides, and 99 putatively novel analogues — and export counts to a report CSV.

## Related tools

- **Cytoscape** (Visualization and layout of the molecular network so molecular families appear as connected components for manual annotation against MIBiG references)
- **BiG-SCAPE** (Generates BGC sequence similarity networks and GCFs from antiSMASH/MIBiG inputs that are matched to molecular families via metabologenomic correlation) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **CORASON** (Resolves evolutionary relationships within GCFs linked to the molecular family, supporting prioritization of the detoxin/rimosamide BGCs) — https://github.com/nselem/corason

## Evaluation signals

- Exported network exists in graphml format and molecular families resolve as connected components in Cytoscape
- Family report reproduces the 110-member detoxin/rimosamide family with the 8/4/99 composition breakdown
- Ion identity calls honor the study filters: mass within 4 ppm, fragmentation cosine similarity above 0.75, ions detected in two or more strains (5,824 ions expected at study scale)
- Known reference compounds map back to the correct MIBiG accessions BGC0001840, BGC0001878 and BGC0001841
- MS/MS preprocessing reproduces the removal of the 25% lowest-intensity ions per scan before network construction

## Limitations

- Molecular network family identification was performed manually — 'The network was manually analyzed to identify ions related to known detoxins and rimosamides' — so annotations are not fully automated and are curator-dependent
- MS/MS data for non-detoxin-producing strains were not deposited in MassIVE and are available only upon request, limiting full reproduction of strain coverage
- LC-MS acquisition parameters were occasionally adjusted (e.g. to target major unnatural isotope ions for optimal fragmentation) rather than fully standardized across the 363 strains

## Evidence

- [readme] BiG-SCAPE role in producing GCF networks from MIBiG references: [evidence span withheld — anchor status: external_doc]
