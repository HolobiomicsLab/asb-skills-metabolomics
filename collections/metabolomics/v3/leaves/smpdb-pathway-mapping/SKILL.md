---
name: smpdb-pathway-mapping
description: Use when a harmonized (or unharmonized) urinary/clinical metabolite compound
  list needs pathway over-representation analysis, especially to compare enrichment
  outcomes with vs without a merging/harmonization strategy.
when_to_use_negative:
- The input is raw spectral data (e.g. mzML) rather than a curated, identifier-resolved
  compound list — MMS harmonization must be completed first.
- The goal is pathway analysis on gene/protein lists rather than metabolite compound
  identifiers.
- A different pathway database than SMPDB is required, since the article's reproducible
  result (two pathways at FDR < 0.05) is specific to MetaboAnalyst 5.0 with SMPDB.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalyst 5.0
  role: Web platform used to run over-representation pathway enrichment on the filtered
    compound lists
- name: SMPDB
  role: Small Molecule Pathway Database supplying the pathway reference set for the
    enrichment analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:7ec681fc5fc45649d25061341a7e77179852c636574fdf51075e0109b02ab65a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 5.0
  - SMPDB
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# smpdb-pathway-mapping

## Summary

Runs over-representation pathway enrichment on a curated metabolite compound list using MetaboAnalyst 5.0 with the Small Molecule Pathway Database (SMPDB), and contrasts results before and after Metabolites Merging Strategy (MMS) harmonization to expose pathways hidden by identifier fragmentation.

## When to use

Use when a harmonized (or unharmonized) urinary/clinical metabolite compound list needs pathway over-representation analysis, especially to compare enrichment outcomes with vs without a merging/harmonization strategy.

## When NOT to use

- The input is raw spectral data (e.g. mzML) rather than a curated, identifier-resolved compound list — MMS harmonization must be completed first.
- The goal is pathway analysis on gene/protein lists rather than metabolite compound identifiers.
- A different pathway database than SMPDB is required, since the article's reproducible result (two pathways at FDR < 0.05) is specific to MetaboAnalyst 5.0 with SMPDB.

## Inputs

- MMS-merged urinary asthma metabolite dataset (391 metabolites, Supplementary Table S2)
- Unmerged dataset of the same compounds (547 entries) without MMS harmonization
- SMPDB pathway library as configured in MetaboAnalyst 5.0

## Outputs

- Per-pathway p-value and FDR for the MMS run
- Per-pathway p-value and FDR for the no-MMS run
- pathway_enrichment_comparison.csv (side-by-side MMS vs no-MMS enrichment table)

## How to apply

Start from the MMS-merged urinary asthma metabolite dataset (391 metabolites, Supplementary Table S2) and filter to compounds found in at least two information sources. Submit the filtered compound list to MetaboAnalyst 5.0 for over-representation analysis against pathways from the Small Molecule Pathway Database (SMPDB). Repeat the identical MetaboAnalyst 5.0 / SMPDB enrichment on the unmerged dataset (547 entries, same compounds without MMS harmonization), also filtered to compounds present in at least two sources. Extract the p-value and false discovery rate (FDR) for every pathway from both runs and assemble a side-by-side comparison table. Judge success by whether the MMS run yields exactly two significant pathways at FDR < 0.05 (tryptophan metabolism and beta-alanine metabolism, with seven pathways relevant at p < 0.05) while the unmerged run yields three relevant (p < 0.05) but zero significant pathways — demonstrating that MMS harmonization reveals pathways hidden by identifier fragmentation.

## Related tools

- **MetaboAnalyst 5.0** (Web platform used to run over-representation pathway enrichment on the filtered compound lists)
- **SMPDB** (Small Molecule Pathway Database supplying the pathway reference set for the enrichment analysis)

## Evaluation signals

- MMS run yields exactly two significant pathways at FDR < 0.05: tryptophan metabolism and beta-alanine metabolism.
- MMS run yields seven pathways relevant at p < 0.05.
- No-MMS run yields three pathways relevant at p < 0.05 but zero pathways reaching FDR < 0.05.
- Comparison table contains p-value and FDR for every pathway from both runs, saved as pathway_enrichment_comparison.csv.
- Compound lists are filtered to compounds found in at least two information sources before enrichment.

## Limitations

- Without a dataset merging strategy such as MMS, significant pathways can remain hidden, so conclusions drawn from unmerged compound lists may be falsely negative.
- Results depend on identifier harmonization quality across non-systematic identifiers (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number).
- Metabolomics studies need standardized and unified metabolite datasets for reproducibility and interstudy comparability; ad-hoc enrichment on heterogeneous lists is not comparable.

## Evidence

- [abstract] MMS applied to urinary asthma metabolites identified significant pathways that were hidden when no dataset merging strategy was followed: [evidence span withheld — anchor status: not_found]
- [abstract] non-systematic identifiers: "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [other] workflow from source card: [evidence span withheld — anchor status: not_found]
- [other] validation criterion: [evidence span withheld — anchor status: not_found]
