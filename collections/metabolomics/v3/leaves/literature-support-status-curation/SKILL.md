---
name: literature-support-status-curation
description: Use when MINNO boundary-flux analysis has flagged reactions missing from
  the KEGG database and you must classify each as publication-supported (e.g. the
  six confirmed B. burgdorferi purine reactions) or homology-backed.
when_to_use_negative:
- When reactions are already validated with literature support flags in an existing
  curated database rather than being newly inferred from MINNO boundary-flux analysis.
- When the goal is de novo reaction prediction or pathway reconstruction rather than
  curating the literature-support status of a pre-existing gap list.
- When the input lacks the species/pathway context (e.g. a generic reaction list with
  no Borrelia-specific provenance), as the curation depends on the article's species-specific
  gap breakdown.
edam_operation: http://edamontology.org/operation_0550
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: MINNO
  role: Source of the missing-reaction gap inventory and boundary-flux evidence used
    to flag reactions absent from KEGG
  repo: github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Homology database used to justify reactions that lack primary-literature confirmation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:e8a7609e973b3a112c1f18eb55d3c9950f1d4700e62eb300dce9829460ba767d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - MINNO
  - NCBI-RefSeq
  edam_operation: http://edamontology.org/operation_0550
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# literature-support-status-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates the literature-support status of metabolite reactions flagged as missing from KEGG by MINNO, separating reactions confirmed by primary literature from those justified by NCBI-RefSeq homology, and normalizes them into a species/pathway-annotated table.

## When to use

Use when MINNO boundary-flux analysis has flagged reactions missing from the KEGG database and you must classify each as publication-supported (e.g. the six confirmed B. burgdorferi purine reactions) or homology-backed.

## When NOT to use

- When reactions are already validated with literature support flags in an existing curated database rather than being newly inferred from MINNO boundary-flux analysis.
- When the goal is de novo reaction prediction or pathway reconstruction rather than curating the literature-support status of a pre-existing gap list.
- When the input lacks the species/pathway context (e.g. a generic reaction list with no Borrelia-specific provenance), as the curation depends on the article's species-specific gap breakdown.

## Inputs

- Supporting Information Table S1 (missing reaction inventory from the MINNO article)
- MINNO GitHub deposit data files from github.com/LewisResearchGroup/MINNO
- KEGG reaction/pathway identifiers for purine, pyrimidine, and nicotinate metabolism

## Outputs

- missing_reactions_table.csv with one row per missing reaction, annotated by species, pathway class, and literature-support flag

## How to apply

Parse Supporting Information Table S1 from the MINNO article (doi:10.1021/acs.analchem.3c04501) and the MINNO GitHub deposit (github.com/LewisResearchGroup/MINNO) into one row per missing reaction, capturing reaction identifiers, pathway assignment (purine, pyrimidine, nicotinate), and the species in which the reaction was found missing (B. burgdorferi, B. parkeri, B. turicatae). Annotate each row with a primary-literature support flag: reactions with explicit publication confirmation get 'literature-supported' (e.g. six confirmed B. burgdorferi purine reactions and four B. burgdorferi pyrimidine reactions, all literature-backed), while the remainder are marked 'NCBI-RefSeq homology' based on the article's statement that 'The remaining reactions show good homology as in the NCBI-RefSeq database'. Cross-check the compiled counts against the reported reaction-gap breakdown: 8 purine reactions for B. burgdorferi, 6 purine reactions shared by B. parkeri/B. turicatae, 4 pyrimidine reactions for B. burgdorferi, 2 pyrimidine reactions shared by B. parkeri/B. turicatae, and 1 nicotinate reaction shared by all three, totaling 18 missing reactions with 9 literature-supported. Write the normalized table to missing_reactions_table.csv. The rationale is to keep the empirical gap inventory auditable by distinguishing reactions validated by experimental evidence from those inferred from sequence similarity.

## Related tools

- **MINNO** (Source of the missing-reaction gap inventory and boundary-flux evidence used to flag reactions absent from KEGG) — github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Homology database used to justify reactions that lack primary-literature confirmation)

## Examples

```
curl -o table_s1.csv 'https://github.com/LewisResearchGroup/MINNO/raw/main/Table_S1.csv' && python -c "import pandas as pd; df=pd.read_csv('table_s1.csv'); print(df[['species','pathway','literature_support']].groupby(['species','pathway']).size())"
```

## Evaluation signals

- Total row count equals 18 missing reactions as reported in the article abstract.
- Exactly 9 rows carry a literature-supported flag, matching the abstract statement.
- Pathway labels resolve cleanly to the three reported classes (purine, pyrimidine, nicotinate) with no unassigned rows.
- Species columns match the reported breakdown: 8 purine gaps in B. burgdorferi, 6 purine gaps shared by B. parkeri and B. turicatae, 4 pyrimidine gaps in B. burgdorferi, 2 pyrimidine gaps shared by B. parkeri and B. turicatae, and 1 nicotinate gap shared by all three.
- No reaction is dual-flagged as both literature-supported and NCBI-RefSeq-homology-only.

## Limitations

- Literature-support status depends on the primary literature already cited in Table S1; reactions confirmed after the article's cutoff will not be reflected.
- Reactions lacking literature support are only justified by NCBI-RefSeq homology, which is a weaker evidence class than experimental confirmation.
- Table S1 is not included in the article body and must be retrieved separately from the Supporting Information or GitHub deposit; no version number for the MINNO deposit is stated.

## Evidence
