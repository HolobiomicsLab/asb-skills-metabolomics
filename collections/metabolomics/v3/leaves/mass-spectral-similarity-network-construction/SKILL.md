---
name: mass-spectral-similarity-network-construction
description: Use when building metabolomic networks from GC-TOF-MS (or similar) feature
  spectra where many molecular features lack structural annotation and you want to
  connect unknowns to identified metabolites via spectral similarity.
when_to_use_negative:
- Features already have structural annotations and KEGG RPAIR biochemical (substrate-product)
  transformations alone would connect them — biochemical/structural networks may suffice
- Input is empirical abundance data across samples rather than spectra — use Empirical
  Dependency Networks (Pearson, biweight or Spearman correlations) instead
- You need spectral library identification of unknowns rather than similarity-based
  edge construction between features
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetaMapR
  role: R/Shiny tool that calculates pairwise spectral similarity networks from uploaded
    mass spectral data and exports edge lists
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR is implemented; used for computing
    cosine correlations and exporting tables
- name: Shiny
  role: R package providing MetaMapR's interactive user interface for tuning thresholds
    and network options
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:d737d40d65dce5d1f9a91a185c8f89422dcefe888e9346d06802ea7bdb3ae617
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectral-similarity-network-construction

## Summary

Constructs spectral similarity networks in MetaMapR by computing pairwise cosine correlations between mass spectra encoded as m/z–intensity pairs and retaining edges above a cosine correlation threshold (>0.7 in the paper), linking structurally unknown metabolites to annotated species.

## When to use

Use when building metabolomic networks from GC-TOF-MS (or similar) feature spectra where many molecular features lack structural annotation and you want to connect unknowns to identified metabolites via spectral similarity.

## When NOT to use

- Features already have structural annotations and KEGG RPAIR biochemical (substrate-product) transformations alone would connect them — biochemical/structural networks may suffice
- Input is empirical abundance data across samples rather than spectra — use Empirical Dependency Networks (Pearson, biweight or Spearman correlations) instead
- You need spectral library identification of unknowns rather than similarity-based edge construction between features

## Inputs

- Mass spectral data table where each molecular feature's spectrum is encoded as mass-to-charge ratio (m/z) and intensity pairs (e.g. from GC-TOF-MS measurements)
- Optional metabolite identifier table (metabolite names or IDs) uploaded to MetaMapR

## Outputs

- Spectral similarity network edge list exported as a comma separated value (.csv) file with cosine correlation scores
- Interactive network visualization with nodes/edges for retained spectral similarity relationships

## How to apply

Load the mass spectral data into MetaMapR (R/Shiny tool), where each molecular feature's spectrum is encoded as mass-to-charge ratio (m/z) and intensity pairs. Compute pairwise cosine correlations between all features' mass spectra, yielding similarity scores bounded between 0 and 1. Filter edges using the cosine correlation threshold >0.7 as applied for Figures 1B and 1C, optionally controlling total edges per object and object-specific edge acceptance (e.g. limiting connections to annotated-to-unknown relationships). Note that MetaMapR handles multiple edge types with a hierarchy Biochemical > Structural > Mass Spectral > Correlation when the 'unique edges' checkbox is selected. Export the retained spectral similarity edge list as a .csv file, which can then be imported into Cytoscape for visualization with node attributes mapped.

## Related tools

- **MetaMapR** (R/Shiny tool that calculates pairwise spectral similarity networks from uploaded mass spectral data and exports edge lists) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR is implemented; used for computing cosine correlations and exporting tables)
- **Shiny** (R package providing MetaMapR's interactive user interface for tuning thresholds and network options)

## Examples

```
Rscript -e "install_github('dgrapov/MetaMapR'); library(shiny); runGitHub('MetaMapR','dgrapov')"  # then upload spectral data, set cosine correlation threshold >0.7, export edge list as .csv
```

## Evaluation signals

- All retained cosine correlation scores fall within the bounds 0 to 1
- Every retained spectral similarity edge exceeds the cosine correlation threshold of >0.7 (as used for Figures 1B and 1C)
- Output edge list exists as a valid .csv file and numeric node IDs map to the row order of the uploaded data (ID 1 = first row analyte, per README FAQ)
- When the 'unique edges' option is selected, no feature retains both a mass spectral edge and a higher-hierarchy (Biochemical > Structural) edge to the same partner
- Edge list imports into Cytoscape and unknown features connect to structurally annotated metabolites as intended

## Limitations

- Threshold values (>0.7 cosine correlation) were chosen for the specific figures shown; the KEGG database limits description of directionality of biochemical transformations, and threshold tuning is user-controlled
- Mass spectral similarity edges link structurally unknown metabolites but do not themselves provide structural annotation
- Requires spectra in m/z–intensity pair form; features lacking usable spectra cannot contribute spectral edges

## Evidence

- [methods] mass spectra encoding: [evidence span withheld — anchor status: not_found]
- [results] unknown extension: [evidence span withheld — anchor status: not_found]
- [readme] edge hierarchy: [evidence span withheld — anchor status: external_doc]
