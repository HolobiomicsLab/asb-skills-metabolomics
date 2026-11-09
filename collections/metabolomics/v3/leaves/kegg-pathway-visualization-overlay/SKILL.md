---
name: kegg-pathway-visualization-overlay
description: Use when a metabolomics analysis needs a visual, pathway-level rendering
  of metabolite abundance changes on the canonical KEGG diagram after metabolites
  have been mapped to KEGG pathway IDs.
when_to_use_negative:
- When the goal is to quantify metabolite contributions to pathway features — use
  Lilikoi's metabolite-pathway regression (lilikoi.meta_path) instead of visualization.
- When metabolites have not yet been mapped to KEGG/HMDB pathway identifiers (no Metabolite_pathway_table
  available from lilikoi.MetaTOpathway/PDSfun).
- When the pathway of interest has no KEGG map or the target organism is not available
  in KEGG (the function relies on KEGG species codes such as 'hsa').
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: Lilikoi v2.0
  role: R package providing the pathway visualization module; lilikoi.KEGGplot wraps
    Pathview to overlay metabolite values on the KEGG diagram
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which Lilikoi v2.0 and the KEGG plotting workflow
    are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:1c7990a31902f530bb5ee1696d6473c315bb3c012945eaeb267d94d1f7eb3f32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kegg-pathway-visualization-overlay

## Summary

Generates a Pathview-style KEGG pathway diagram in Lilikoi v2.0, overlaying metabolite-level values from a metabolomics dataset (e.g., 207-sample plasma breast cancer PR000284 workbench data) onto KEGG pathway maps such as alanine-aspartate-glutamate metabolism (KEGG pathid 00250).

## When to use

Use when a metabolomics analysis needs a visual, pathway-level rendering of metabolite abundance changes on the canonical KEGG diagram after metabolites have been mapped to KEGG pathway IDs.

## When NOT to use

- When the goal is to quantify metabolite contributions to pathway features — use Lilikoi's metabolite-pathway regression (lilikoi.meta_path) instead of visualization.
- When metabolites have not yet been mapped to KEGG/HMDB pathway identifiers (no Metabolite_pathway_table available from lilikoi.MetaTOpathway/PDSfun).
- When the pathway of interest has no KEGG map or the target organism is not available in KEGG (the function relies on KEGG species codes such as 'hsa').

## Inputs

- Metabolite abundance matrix (e.g., plasma_breast_cancer.csv dataSet from PR000284 workbench, 207 samples)
- Metabolite-to-pathway mapping table from lilikoi.MetaTOpathway (Metabolite_pathway_table)
- Sample phenotype/label metadata (sampleinfo) and desired group ordering
- KEGG pathway id (e.g., '00250') and species code (e.g., 'hsa')

## Outputs

- Pathview-rendered KEGG pathway image file for the specified pathid, with metabolite-level values overlaid, suffixed per filesuffix (e.g., 'GSE16873')

## How to apply

After the Lilikoi feature mapper (lilikoi.MetaTOpathway 'name' → lilikoi.PDSfun) converts metabolite names to HMDB IDs and to pathways, subset the metabolite matrix to entries mapped to the pathway of interest — for the alanine-aspartate-glutamate pathway this is KEGG pathid 00250. Call lilikoi.KEGGplot, supplying the metabolite abundance matrix (metamat), the sample phenotype information (sampleinfo) and group order, the pathid ('00250'), the species ('hsa'), and the Metabolite_pathway_table produced by the mapping step. The function overlays metabolite-level values onto the KEGG pathway diagram via the Pathview module. Rationale: Lilikoi v2.0 explicitly added pathway visualization to augment pathway-based metabolomics analysis, letting researchers see where significant metabolites (e.g., citric acid, pyruvate, glutamine, oxaloacetate, asparagine elevated in ER− disease, P < 0.05) act on the pathway graph.

## Related tools

- **Lilikoi v2.0** (R package providing the pathway visualization module; lilikoi.KEGGplot wraps Pathview to overlay metabolite values on the KEGG diagram) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which Lilikoi v2.0 and the KEGG plotting workflow are executed)

## Examples

```
lilikoi.KEGGplot(metamat = metamat, sampleinfo = sampleinfo, grouporder = grouporder, pathid = '00250', specie = 'hsa', filesuffix = 'GSE16873', Metabolite_pathway_table = Metabolite_pathway_table)
```

## Evaluation signals

- A Pathview image for pathid 00250 is actually generated (file exists with the expected filesuffix, e.g., 'GSE16873').
- The overlaid values correspond to metabolites mapped to the alanine-aspartate-glutamate pathway via Metabolite_pathway_table (subset consistent with KEGG pathid 00250, species hsa).
- Sample group ordering in the plot matches sampleinfo/metadata labels (e.g., ER− vs ER+ breast cancer groups).
- The plot is reproducible from the saved PDS matrix and mapping table outputs of the preceding Lilikoi feature-mapper steps.
- Metabolites highlighted in the figure align with the regression findings (citric acid, pyruvate, glutamine, oxaloacetate, asparagine significant at P < 0.05 in ER− disease).

## Limitations

- Depends on KEGG availability of the pathway and species; mapping quality relies on the Lilikoi MetaTOpathway name-to-HMDB conversion step.
- Visualization alone does not establish statistical significance — significance claims come from the companion single-variate metabolite-pathway regression (P < 0.05 threshold).
- GigaDB accession/DOI for code and data snapshots is not given in the available sections, so reproducibility may require retrieving GigaDB separately.

## Evidence

- [abstract] Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression.: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [intro] Third, we augmented the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization.: "Third, we augmented the pathway-based metabolomics analysis with metabolite-pathway regression and pathway visualization."
- [readme] lilikoi.KEGGplot(metamat = metamat, sampleinfo = sampleinfo, grouporder = grouporder, pathid = '00250', specie = 'hsa': "lilikoi.KEGGplot(metamat = metamat, sampleinfo = sampleinfo, grouporder = grouporder,
                 pathid = '00250', specie = 'hsa',
                 filesuffix = 'GSE16873'"
- [readme] Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function: "# Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function"
