---
name: ms2lda-mass2motif-mapping
description: Use when annotated MS1/MS2 peak features should be grouped into GNPS
  FBMN Molecular Families and MS2LDA Mass2Motif metabolite sets for PLAGE-based activity
  scoring in PALS.
when_to_use_negative:
- The dataset has no MS/MS fragmentation spectra, so no GNPS FBMN molecular families
  or MS2LDA motifs can be derived.
- The goal is strictly canonical pathway analysis of already-annotated compounds —
  then direct KEGG (PiMP_KEGG) or Reactome database queries suffice.
- The input is a proteomics or transcriptomics table rather than metabolomics peak
  data (Reactome UniProt/ENSEMBL modes, not MF/Motif sets, apply).
edam_operation: null
tools:
- name: PALS
  role: Python library/command-line tool that consumes MF and Mass2Motif metabolite
    sets and ranks them via PLAGE; PALS Viewer prioritises MF/Motifs by activity level
  repo: https://github.com/glasgowcompbio/PALS
- name: GNPS
  role: Source of Molecular Families grouping peaks by shared fragmentation spectra
- name: MS2LDA
  role: Source of Mass2Motifs that group peaks by shared substructure motifs
- name: pandas
  role: DataFrame format in which MF/Motif mapping tables and intensity data are passed
    to PALS programmatically
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:562cc06ea2acc84e35c76703d6bdbb239c29d7163796852b643097cfeb6140da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - GNPS
  - MS2LDA
  - PALS
  - Python
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ms2lda-mass2motif-mapping

## Summary

Builds user-defined metabolite sets from GNPS Molecular Families and MS2LDA Mass2Motifs so that unannotated, fragmentation-grouped peaks can be ranked by activity level alongside canonical KEGG/Reactome pathways in PALS.

## When to use

Use when annotated MS1/MS2 peak features should be grouped into GNPS FBMN Molecular Families and MS2LDA Mass2Motif metabolite sets for PLAGE-based activity scoring in PALS.

## When NOT to use

- The dataset has no MS/MS fragmentation spectra, so no GNPS FBMN molecular families or MS2LDA motifs can be derived.
- The goal is strictly canonical pathway analysis of already-annotated compounds — then direct KEGG (PiMP_KEGG) or Reactome database queries suffice.
- The input is a proteomics or transcriptomics table rather than metabolomics peak data (Reactome UniProt/ENSEMBL modes, not MF/Motif sets, apply).

## Inputs

- Intensity matrix of peak features (rows = peak IDs, columns = samples, with group line or experimental_design dict)
- Annotation matrix mapping peak IDs to KEGG or ChEBI compound IDs (peaks without annotations excluded)
- GNPS FBMN molecular family / quantification table
- MS2LDA Mass2Motif decomposition results

## Outputs

- Feature-to-set mapping tables linking each annotated peak ID to its Molecular Family and Mass2Motif IDs
- Activity-ranked metabolite-set table (pathway IDs with p-value columns, computed via PLAGE/ORA/GSEA)

## How to apply

Retrieve molecular families from a GNPS FBMN analysis and Mass2Motifs from an MS2LDA decomposition in Python, following the GNPS_analysis.ipynb notebook workflow. Convert these groupings into feature-to-set mapping tables where each annotated peak ID is linked to its Molecular Family and/or Mass2Motif ID; note that peaks without annotations are excluded, so peaks grouped in a MF/Motif but missing a KEGG or ChEBI compound ID still participate via their co-fragmented members. Programmatic use passes these as pandas DataFrames into a PALS DataSource object, after which any metabolite-set ranking method (PLAGE, ORA, GSEA) can score the sets. Validation: mapping tables must reference only peak IDs present in the annotation matrix, and set IDs must come from the chosen GNPS/MS2LDA output; in PALS Viewer, MF and Mass2Motifs can then be prioritised by their activity levels.

## Related tools

- **PALS** (Python library/command-line tool that consumes MF and Mass2Motif metabolite sets and ranks them via PLAGE; PALS Viewer prioritises MF/Motifs by activity level) — https://github.com/glasgowcompbio/PALS
- **GNPS** (Source of Molecular Families grouping peaks by shared fragmentation spectra)
- **MS2LDA** (Source of Mass2Motifs that group peaks by shared substructure motifs)
- **pandas** (DataFrame format in which MF/Motif mapping tables and intensity data are passed to PALS programmatically)

## Examples

```
from pals.PLAGE import PLAGE; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, database_name, min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- Every peak ID in the MF/Motif mapping tables is present in the annotation matrix row IDs (unannotated peaks excluded).
- All set IDs in the mapping tables are valid Molecular Family or Mass2Motif identifiers drawn from the supplied GNPS/MS2LDA outputs.
- The ranked output table exposes pathway IDs sortable by p-value columns, with unq_pw_F, tot_ds_F, and F_coverage columns populated.
- Programmatic path reproduces the README contract: a DataSource built from int_df/annotation_df/experimental_design yields a pathway DataFrame via method.get_pathway_df().

## Limitations

- Multiple peak IDs may map to multiple compound IDs and vice versa due to identification uncertainty, complicating set membership.
- Peaks lacking compound annotations never contribute to pathway analysis, so MF/Motif coverage is limited to annotated members plus their co-grouped peaks.
- Groupings depend entirely on the quality of the upstream GNPS FBMN and MS2LDA analyses; noisy or missing peaks affect set composition.
- Minimum-intensity imputation (default min_replace=5000) and log-2/standardisation are applied before scoring and affect set activity levels.

## Evidence

- [readme] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed.: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed."
- [readme] you can also prioritise MF and Mass2Motifs from your GNPS analysis based on their activity levels.: [evidence span withheld — anchor status: external_doc]
- [methods] analysing potentially unknown metabolites that have been grouped into Molecular Families (by GNPS) and into Mass2Motifs (by MS2LDA): "analysing potentially unknown metabolites that have been grouped into Molecular Families (by GNPS) and into Mass2Motifs (by MS2LDA)"
- [intro] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra: [evidence span withheld — anchor status: not_found]
