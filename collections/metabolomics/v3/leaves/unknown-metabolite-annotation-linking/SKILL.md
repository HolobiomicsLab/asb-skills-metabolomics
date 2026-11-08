---
name: unknown-metabolite-annotation-linking
description: Use when a metabolomics feature table contains unannotated peaks (unknowns)
  lacking KEGG/PubChem identifiers, and the goal is to connect them to annotated compounds
  using mass spectral similarity rather than enzymatic or structural relationships.
when_to_use_negative:
- The metabolites of interest already have KEGG or PubChem CIDs and can be connected
  via KEGG RPAIR biochemical transformation networks instead
- The goal is structural similarity between known compounds (use PubChem Substructure
  Fingerprints with Tanimoto score >0.7, the paper's structural network threshold)
- The data are non-spectral empirical measurements (e.g. clinical covariates) where
  Pearson/biweight or Spearman empirical dependency networks, not cosine spectral
  similarity, are appropriate
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaMapR
  role: Calculates spectral similarity networks from pairwise mass spectral comparisons,
    applies the cosine correlation threshold and edge-acceptance controls, and exports
    the edge list
  repo: https://github.com/dgrapov/Meta MapR
- name: R
  role: Programming language in which MetaMapR (Shiny-based) is implemented and in
    which the cosine-correlation edge filtering and .csv export are performed
  repo: http://cran.us.r-project.org/
- name: Shiny server
  role: Hosts MetaMapR as a standalone web application for interactive network calculation
    and threshold tuning
  repo: https://github.com/rstudio/shiny-server
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:71a7f53582e4c5f85b07d7f938a33717356f3f3da9cc1101a74fc515b90becae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/Meta MapR
  tools:
  - MetaMapR
  - R
  - Shiny server
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Unknown-metabolite annotation linking

## Summary

Links structurally unknown metabolites into a metabolic network via pairwise mass spectral similarity (cosine correlation) edges, as implemented in MetaMapR's spectral similarity network module, so that unannotated GC-TOF-MS features inherit biochemical context from identified neighbors.

## When to use

Use when a metabolomics feature table contains unannotated peaks (unknowns) lacking KEGG/PubChem identifiers, and the goal is to connect them to annotated compounds using mass spectral similarity rather than enzymatic or structural relationships.

## When NOT to use

- The metabolites of interest already have KEGG or PubChem CIDs and can be connected via KEGG RPAIR biochemical transformation networks instead
- The goal is structural similarity between known compounds (use PubChem Substructure Fingerprints with Tanimoto score >0.7, the paper's structural network threshold)
- The data are non-spectral empirical measurements (e.g. clinical covariates) where Pearson/biweight or Spearman empirical dependency networks, not cosine spectral similarity, are appropriate

## Inputs

- Mass spectral data table from a MetaMapR example dataset, with each molecular feature's spectrum encoded as m/z and intensity pairs
- Feature annotation status (identified vs. structurally unknown metabolites)
- Optional network edge-acceptance options (total edges per object, annotated-to-unknown-only links)

## Outputs

- Spectral similarity network edge list (node pairs with cosine correlation scores >0.7) exported as a .csv file
- Network importable into Cytoscape for interactive visualization and node attribute mapping

## How to apply

Encode each molecular feature's mass spectrum as mass-to-charge ratio (m/z) and intensity pairs, then compute pairwise cosine correlations between all spectra, yielding similarity scores bounded between 0 and 1. Filter spectral similarity edges using the cosine correlation threshold >0.7 used for Figures 1B and 1C in the MetaMapR T1D NOD mouse study (~470 plasma metabolites measured by GC-TOF-MS). Optionally constrain the network with controls of total edges per object and object-specific edge acceptance, e.g. limiting connections to annotated-to-unknown relationships. When combining edge types, MetaMapR applies a hierarchy Biochemical > Structural > Mass Spectral > Correlation if the 'unique edges' option is selected. Export the retained edge list as a comma separated value (.csv) file for import into Cytoscape; numeric node IDs map to uploaded data rows in order.

## Related tools

- **MetaMapR** (Calculates spectral similarity networks from pairwise mass spectral comparisons, applies the cosine correlation threshold and edge-acceptance controls, and exports the edge list) — https://github.com/dgrapov/Meta MapR
- **R** (Programming language in which MetaMapR (Shiny-based) is implemented and in which the cosine-correlation edge filtering and .csv export are performed) — http://cran.us.r-project.org/
- **Shiny server** (Hosts MetaMapR as a standalone web application for interactive network calculation and threshold tuning) — https://github.com/rstudio/shiny-server

## Evaluation signals

- The output edge list exists as a .csv file
- All retained cosine correlation scores fall within 0 to 1
- Every retained edge exceeds the 0.7 cosine correlation threshold used for Figures 1B and 1C
- Unknown (structurally unannotated) features are connected to identified species in the network, mirroring Figure 1B's use of mass spectral similarity
- Numeric node IDs in the edge list map one-to-one to rows of the uploaded data (or a custom unique edge index supplied under network > more options)

## Limitations

- Mass spectral similarity links unknowns by spectral resemblance only; it does not imply biochemical transformation, so edges from this module are lower priority than biochemical edges in the unique-edges hierarchy
- The 0.7 cosine correlation cutoff was chosen for specific figures (1B and 1C); users tuning the threshold change which unknowns get linked
- Networks exported from MetaMapR required further enhancement in Cytoscape (mapping empirical and domain-knowledge variables to node attributes); the raw edge list alone is not the final figure
- No changelog found in the repository discussion section, so version behavior of the spectral module is undocumented

## Evidence

- [results] Mass spectral extends analysis to unknowns: [evidence span withheld — anchor status: not_found]
- [readme] Unique edges hierarchy in README: [evidence span withheld — anchor status: external_doc]
