---
name: gnps-molecular-family-mapping
description: Use when a metabolomics intensity matrix contains peaks grouped by tandem-MS
  fragmentation spectra (GNPS FBMN Molecular Families, MS2LDA Mass2Motifs) and these
  groupings must be turned into metabolite-set membership tables for PALS activity-level
  scoring.
when_to_use_negative:
- When every peak already carries trustworthy KEGG/ChEBI annotations and curated pathway
  membership suffices — use PiMP_KEGG or Reactome (COMPOUND/ChEBI) database queries
  instead
- When the dataset has no fragmentation-spectrum grouping outputs (no GNPS FBMN or
  MS2LDA results) to derive metabolite sets from
- When the goal is gene/protein pathway analysis — Reactome UniProt/ENSEMBL modes,
  not MF/Mass2Motif mapping, cover proteomics and transcriptomics
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PALS
  role: consumes the MF/Mass2Motif metabolite-set mapping tables and scores them via
    the PLAGE decomposition; also exposes the GNPS_analysis.ipynb notebook workflow
  repo: https://github.com/glasgowcompbio/PALS
- name: PALS Viewer
  role: prioritises Molecular Families and Mass2Motifs from GNPS analysis based on
    their activity levels
  repo: https://pals.glasgowcompbio.org/app/
- name: GNPS
  role: source of Molecular Family groupings from molecular networking / feature-based
    molecular networking
- name: MS2LDA
  role: source of Mass2Motif substructure annotations retrieved as metabolite sets
- name: pandas
  role: holds the intensity and annotation data frames from which peak-to-set mapping
    tables are assembled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:f2a7ba3cecdde036b0a2217708a6c5e0a6268c3915594d3721e908ddae20c8ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - GNPS
  - MS2LDA
  - PALS
  - PALS Viewer
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# GNPS Molecular Family Mapping

## Summary

Builds user-defined metabolite sets from fragmentation-spectrum groupings — GNPS Molecular Families (FBMN) and MS2LDA Mass2Motifs — as peak-to-set mapping tables that PALS can score with the PLAGE decomposition, extending pathway analysis to unknown metabolites.

## When to use

Use when a metabolomics intensity matrix contains peaks grouped by tandem-MS fragmentation spectra (GNPS FBMN Molecular Families, MS2LDA Mass2Motifs) and these groupings must be turned into metabolite-set membership tables for PALS activity-level scoring.

## When NOT to use

- When every peak already carries trustworthy KEGG/ChEBI annotations and curated pathway membership suffices — use PiMP_KEGG or Reactome (COMPOUND/ChEBI) database queries instead
- When the dataset has no fragmentation-spectrum grouping outputs (no GNPS FBMN or MS2LDA results) to derive metabolite sets from
- When the goal is gene/protein pathway analysis — Reactome UniProt/ENSEMBL modes, not MF/Mass2Motif mapping, cover proteomics and transcriptomics

## Inputs

- pandas intensity matrix (int_df) of peak features vs samples from metabolomics peak data
- annotation matrix mapping peak IDs to KEGG or ChEBI compound IDs
- GNPS FBMN Molecular Family assignments (fragmentation-spectrum groupings)
- MS2LDA Mass2Motif annotations from ms2lda.org

## Outputs

- Feature-to-Molecular-Family mapping table (peak ID -> MF ID)
- Feature-to-Mass2Motif mapping table (peak ID -> Motif ID)
- User-defined metabolite sets usable by PALS PLAGE decomposition and prioritisation in PALS Viewer

## How to apply

Retrieve Molecular Family assignments from GNPS FBMN outputs and Mass2Motif annotations from MS2LDA in Python, following the GNPS_analysis.ipynb notebook workflow shipped with PALS. Load the annotation matrix with pandas so that each peak ID (row index) is paired with its KEGG or ChEBI annotation, then assemble feature-to-set tables linking each peak ID to its Molecular Family ID and/or Mass2Motif ID. Use these mapping tables as user-defined metabolite sets passed into the PALS DataSource/PLAGE machinery alongside the intensity matrix, since the decomposition approach is amenable to any group of metabolite sets, not just pathways. Validation: mapping tables must reference only peak IDs present in the intensity/annotation matrices, and set IDs must come only from the GNPS/MS2LDA exports; from PALS Viewer, MF and Mass2Motifs can then be prioritised by their activity levels.

## Related tools

- **PALS** (consumes the MF/Mass2Motif metabolite-set mapping tables and scores them via the PLAGE decomposition; also exposes the GNPS_analysis.ipynb notebook workflow) — https://github.com/glasgowcompbio/PALS
- **PALS Viewer** (prioritises Molecular Families and Mass2Motifs from GNPS analysis based on their activity levels) — https://pals.glasgowcompbio.org/app/
- **GNPS** (source of Molecular Family groupings from molecular networking / feature-based molecular networking)
- **MS2LDA** (source of Mass2Motif substructure annotations retrieved as metabolite sets)
- **pandas** (holds the intensity and annotation data frames from which peak-to-set mapping tables are assembled)

## Evaluation signals

- Every peak ID appearing in the MF/Mass2Motif mapping tables exists in the annotation/intensity matrices — peaks without annotations are excluded from pathway analysis
- All set IDs in the mapping tables are Molecular Family or Mass2Motif identifiers traceable to the GNPS/MS2LDA exports, not invented or from KEGG/Reactome
- Each annotated peak with a fragmentation grouping appears in at least one set, and multi-assignment peaks (multiple peak IDs to multiple compounds and vice versa) are retained as repeated rows
- Feeding the mapping into PALS PLAGE yields a pathway ranking table whose rows include the MF/Mass2Motif sets, rankable by p-value columns

## Limitations

- Peaks lacking compound annotations are not used for pathway analysis, so MF/Motif sets built from unannotated features cannot enter the annotation-driven mapping stage
- Annotation uncertainty means multiple peak IDs may map to multiple compound IDs and vice versa, complicating one-to-one set membership assumptions
- Metabolite-set scoring results depend on preprocessing choices — normalisation can significantly impact pathway analysis results

## Evidence

- [readme] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed"
- [methods] Grouping unknown metabolites into Molecular Families and Mass2Motifs: "analysing potentially unknown metabolites that have been grouped into Molecular Families (by GNPS) and into Mass2Motifs (by MS2LDA)"
- [methods] Unannotated peaks excluded: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
- [readme] Annotation ambiguity: "multiple peak IDs may be mapped to multiple compound IDs and \textit{vice versa}"
