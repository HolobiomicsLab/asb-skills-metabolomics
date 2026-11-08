---
name: gene-lipid-class-association
description: Use when a lipid feature abundance matrix plus phenotype file must be
  linked to lipid classes (via genes) and analysed for differential lipid class, total
  chain length and hydrocarbon-chain saturation.
when_to_use_negative:
- The input is a transcriptomic (RNA-seq/miRNA/isoform) count matrix handled by the
  genes, miRNA or isoform modules rather than a lipid abundance matrix.
- No phenotype file with defined contrast groups is available, since edgeR differential
  abundance contrasts depend on it.
- Lipid identifiers cannot be parsed into lipid-class, chain-length and saturation
  components by lipidr, so the three lipid feature dimensions cannot be computed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: LipiDB
  role: annotates lipid species and associates genes with lipid classes present in
    the data via a manually curated database, producing a gene-lipid-class heatmap
- name: lipidr
  role: parses lipid identifiers into lipid-class, total chain length and saturation
    features; computes abundance summaries and differential/enrichment results
- name: edgeR
  role: performs differential abundance analysis on lipid feature dimensions against
    the phenotype groups
- name: preprocessCore
  role: provides the preprocessing utilities behind the mstus normalization applied
    to lipid intensities
- name: ComplexHeatmap
  role: renders the lipid-class differential results heatmap (lipid_class_heatmap.png)
- name: MultiOmicsIntegrator (MOI)
  role: host pipeline whose lipids module packages the LipiDB/lipidr gene-lipid-class
    association workflow
  repo: ASAGlab/moi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:9e3a8450169f4964bf99985231fd87a4d6a795fbb1f3593497e6eebe22dbc002
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-lipid-class-association

## Summary

The lipidomics module of MultiOmicsIntegrator (MOI), in which LipiDB associates genes with lipid classes present in the data from a manually curated database, while lipidr parses lipid identifiers into class, chain-length and saturation dimensions for differential analysis and enrichment.

## When to use

Use when a lipid feature abundance matrix plus phenotype file must be linked to lipid classes (via genes) and analysed for differential lipid class, total chain length and hydrocarbon-chain saturation.

## When NOT to use

- The input is a transcriptomic (RNA-seq/miRNA/isoform) count matrix handled by the genes, miRNA or isoform modules rather than a lipid abundance matrix.
- No phenotype file with defined contrast groups is available, since edgeR differential abundance contrasts depend on it.
- Lipid identifiers cannot be parsed into lipid-class, chain-length and saturation components by lipidr, so the three lipid feature dimensions cannot be computed.

## Inputs

- lipid feature matrix (lipid species abundance/intensity table)
- phenotype file defining the contrast groups for the lipids module
- params.yml configuration for the lipids module

## Outputs

- lipid_class_heatmap.png (genes related to available lipid classes, rendered by ComplexHeatmap)
- differential abundance tables for lipid classes, chain length and degree of saturation (text files)
- intermediate R objects (RData) under the lipids module output directory
- gene-to-lipid-class associations from LipiDB's manually curated database

## How to apply

Load the lipid feature matrix and its accompanying phenotype file as specified in the params.yml of the lipids module, then normalize lipid intensities with the mstus approach implemented via preprocessCore preprocessing utilities, which MOI applies specifically to proteins and lipids. Use lipidr to parse lipid identifiers into lipid-class, total chain length, and saturation features and compute per-feature abundance summaries that associate genes with lipid classes present in the data. Run differential abundance analysis on these lipid feature dimensions with edgeR, contrasting the phenotype groups defined in the phenotype file. Generate the differential results as a lipid-class heatmap with ComplexHeatmap saved as lipid_class_heatmap.png, and write the differential tables and intermediate objects as text files and RData objects under the lipids module directory of the user-defined output folder. LipiDB performs the gene-lipid-class association using a manually curated database, so associations are only as complete as that curation.

## Related tools

- **LipiDB** (annotates lipid species and associates genes with lipid classes present in the data via a manually curated database, producing a gene-lipid-class heatmap)
- **lipidr** (parses lipid identifiers into lipid-class, total chain length and saturation features; computes abundance summaries and differential/enrichment results)
- **edgeR** (performs differential abundance analysis on lipid feature dimensions against the phenotype groups)
- **preprocessCore** (provides the preprocessing utilities behind the mstus normalization applied to lipid intensities)
- **ComplexHeatmap** (renders the lipid-class differential results heatmap (lipid_class_heatmap.png))
- **MultiOmicsIntegrator (MOI)** (host pipeline whose lipids module packages the LipiDB/lipidr gene-lipid-class association workflow) — ASAGlab/moi

## Examples

```
nextflow run ASAGlab/moi -c params_lipids.yml -profile docker
```

## Evaluation signals

- lipid_class_heatmap.png exists in the lipids module output directory
- differential lipidomics tables exist both as text files and RData objects
- the heatmap and tables cover all three lipid feature dimensions: lipid class, chain length, and degree of saturation
- the gene-to-lipid-class heatmap links genes only to lipid classes actually present in the input abundance matrix
- lipid intensities have been normalized with mstus (preprocessCore) before differential analysis

## Limitations

- Gene-lipid-class associations rely on LipiDB's manually curated database, so lipid-class coverage is bounded by that curation.
- mstus normalization is the method MOI applies specifically to proteins and lipids; substituting other normalization schemes deviates from the module's design.
- Differential contrasts are limited to the phenotype groups defined in the accompanying phenotype file.

## Evidence

- [intro] The pipeline performs lipidomics analysis: [evidence span withheld — anchor status: not_found]
- [readme] MOI is a best-practice multi-omics pipeline: "**multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
- [intro] MOI integrates multiple omics datasets: "Integration of multi omics data"
