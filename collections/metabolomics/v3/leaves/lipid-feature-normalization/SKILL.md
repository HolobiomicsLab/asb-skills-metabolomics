---
name: lipid-feature-normalization
description: Use when a lipid feature/abundance matrix plus a phenotype file must
  be normalized (mstus) and decomposed via lipidr into lipid class, chain length,
  and saturation features before differential lipid analysis in MOI.
when_to_use_negative:
- Input is transcriptomics data (mRNA, miRNA, isoform count matrices) — use MOI's
  RNA-seq modules (filterByExp, quantile normalization, edgeR/DESeq2/RankProduct)
  instead.
- Input is already a normalized, lipidr-annotated feature table with no phenotype
  file — differential contrasts against phenotype groups cannot be defined, so skip
  to downstream visualization only.
- The analysis target is multi-omics integration (MCIA PCA) rather than lipid-specific
  normalization and differential analysis.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: lipidr
  role: Parses lipid identifiers into lipid-class, total chain length, and saturation
    features; performs lipid class annotation, differential analysis, and enrichment
    of classes, chain length, and hydrocarbon-chain saturation.
- name: preprocessCore (mstus)
  role: Provides the preprocessing utilities behind the mstus normalization MOI applies
    specifically to proteins and lipids.
- name: edgeR
  role: Runs differential abundance analysis on lipid features (classes, chain lengths,
    saturation states) contrasting phenotype groups from the phenotype file.
- name: ComplexHeatmap
  role: Generates lipid_class_heatmap.png summarizing the differential lipid-class
    results.
- name: LipiDB
  role: Annotates lipid species and associates genes with specific lipid classes present
    in the data using a manually curated database.
- name: MultiOmicsIntegrator (MOI)
  role: Hosts the lipids module that executes this normalization-annotation-differential
    workflow.
  repo: ASAGlab/moi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:135fab9afb38f5805fffff18825a434c212ad098e5b110f7db800c9d4a461573
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - ComplexHeatmap
  - LipiDB
  - MultiOmicsIntegrator (MOI)
  - edgeR
  - lipidr
  - mstus
  - preprocessCore
  - preprocessCore (mstus)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lipid-feature-normalization

## Summary

Normalizes lipid abundance matrices with the mstus approach (preprocessCore), then uses lipidr to parse lipid identifiers into lipid-class, total chain length, and degree-of-saturation features for MOI's lipidomics module.

## When to use

Use when a lipid feature/abundance matrix plus a phenotype file must be normalized (mstus) and decomposed via lipidr into lipid class, chain length, and saturation features before differential lipid analysis in MOI.

## When NOT to use

- Input is transcriptomics data (mRNA, miRNA, isoform count matrices) — use MOI's RNA-seq modules (filterByExp, quantile normalization, edgeR/DESeq2/RankProduct) instead.
- Input is already a normalized, lipidr-annotated feature table with no phenotype file — differential contrasts against phenotype groups cannot be defined, so skip to downstream visualization only.
- The analysis target is multi-omics integration (MCIA PCA) rather than lipid-specific normalization and differential analysis.

## Inputs

- Lipid feature/abundance matrix (lipid species intensities)
- Phenotype file defining sample groups for contrasts
- params.yml for the MOI lipids module

## Outputs

- mstus-normalized lipid intensity matrix
- Differential lipidomics tables (lipid classes, chain length, degree of saturation) as text files
- Intermediate objects saved as RData under the lipids module directory
- lipid_class_heatmap.png generated with ComplexHeatmap

## How to apply

Load the lipid feature matrix and the accompanying phenotype file as specified in the params.yml for the lipids module, then normalize lipid intensities with the mstus normalization approach implemented through preprocessCore — the method MOI applies specifically to proteins and lipids. Parse lipid identifiers with lipidr to derive lipid-class, total chain length, and saturation features and compute per-feature abundance summaries that associate genes with lipid classes present in the data (backed by LipiDB's manually curated database). Run differential abundance analysis with edgeR contrasting the phenotype groups defined in the phenotype file across the three feature dimensions (class, chain length, saturation). Visualize results as a lipid-class heatmap with ComplexHeatmap (lipid_class_heatmap.png) and write the differential tables and intermediate objects as text and RData files under the lipids module directory of the user-defined output folder.

## Related tools

- **lipidr** (Parses lipid identifiers into lipid-class, total chain length, and saturation features; performs lipid class annotation, differential analysis, and enrichment of classes, chain length, and hydrocarbon-chain saturation.)
- **preprocessCore (mstus)** (Provides the preprocessing utilities behind the mstus normalization MOI applies specifically to proteins and lipids.)
- **edgeR** (Runs differential abundance analysis on lipid features (classes, chain lengths, saturation states) contrasting phenotype groups from the phenotype file.)
- **ComplexHeatmap** (Generates lipid_class_heatmap.png summarizing the differential lipid-class results.)
- **LipiDB** (Annotates lipid species and associates genes with specific lipid classes present in the data using a manually curated database.)
- **MultiOmicsIntegrator (MOI)** (Hosts the lipids module that executes this normalization-annotation-differential workflow.) — ASAGlab/moi

## Examples

```
nextflow run ASAGlab/moi -c nf-core/mom/params_lipids.yml -profile docker
```

## Evaluation signals

- lipid_class_heatmap.png exists in the user-defined output folder after the run.
- Differential lipidomics tables exist in both text and RData form under the lipids module directory.
- The heatmap and tables cover all three lipid feature dimensions reported by the module: lipid class, chain length, and degree of saturation.
- Normalized intensities trace to the mstus approach via preprocessCore, matching MOI's lipid/protein-specific normalization choice.
- Differential contrasts correspond exactly to the phenotype groups defined in the phenotype file supplied via params.yml.

## Limitations

- Software versions are not specified in the paper for the lipids module tools.
- LipiDB gene–lipid-class association depends on a manually curated database, so coverage is bounded by curation.
- MOI fixes mstus (via preprocessCore) as the normalization for proteins and lipids; alternative normalization strategies are not offered for the lipids module.

## Evidence

- [full_text] lipidomics mstus normalization: [evidence span withheld — anchor status: not_found]
- [full_text] lipidr feature parsing: [evidence span withheld — anchor status: not_found]
- [full_text] lipidr outputs: [evidence span withheld — anchor status: not_found]
- [intro] pipeline scope: [evidence span withheld — anchor status: not_found]
