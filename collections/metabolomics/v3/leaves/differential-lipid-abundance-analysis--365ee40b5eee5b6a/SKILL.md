---
name: differential-lipid-abundance-analysis
description: Use when the input is a lipid feature (intensity) matrix plus a phenotype
  file and the goal is differential abundance and enrichment across lipid classes,
  chain length, and degree of saturation.
when_to_use_negative:
- Input is an RNA-seq count matrix rather than a lipid feature matrix — gene/isoform
  differential analysis in MOI uses edgeR/DESeq2/RankProduct on transcript data, not
  the lipidomics module.
- Only raw chromatogram/mass-spectrometry files are available with no lipid intensity
  matrix — lipidr feature parsing here operates on an already-quantified lipid feature
  matrix.
- No phenotype file with contrast groups is provided — edgeR differential testing
  in this module requires predefined phenotype groups.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: parses lipid identifiers into lipid-class, chain-length and saturation features,
    annotates lipid classes, and computes per-feature abundance summaries; also drives
    enrichment of lipid classes, chain length, and degree of saturation
- name: edgeR
  role: performs differential abundance testing on lipid features across phenotype
    groups
- name: preprocessCore
  role: provides preprocessing utilities underlying the mstus normalization applied
    to protein and lipid intensities
- name: ComplexHeatmap
  role: renders the lipid-class heatmap of differential results (lipid_class_heatmap.png)
- name: LipiDB
  role: annotates lipid species and associates genes with specific lipid classes present
    in the data using a manually curated database
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline hosting the lipids module, including filtering, normalization
    and differential analysis steps
  repo: https://github.com/ASAGlab/moi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:3be311ddb2620f0ec87a054a9f700969f6d3a1d7c9ac51640e0910773f3f1976
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# differential-lipid-abundance-analysis

## Summary

Lipidomics differential analysis module of the MultiOmicsIntegrator (MOI) Nextflow pipeline: lipid feature matrices are mstus-normalized, parsed into lipid-class/chain-length/saturation features with lipidr, tested for differential abundance with edgeR, and summarized as a lipid-class heatmap.

## When to use

Use when the input is a lipid feature (intensity) matrix plus a phenotype file and the goal is differential abundance and enrichment across lipid classes, chain length, and degree of saturation.

## When NOT to use

- Input is an RNA-seq count matrix rather than a lipid feature matrix — gene/isoform differential analysis in MOI uses edgeR/DESeq2/RankProduct on transcript data, not the lipidomics module.
- Only raw chromatogram/mass-spectrometry files are available with no lipid intensity matrix — lipidr feature parsing here operates on an already-quantified lipid feature matrix.
- No phenotype file with contrast groups is provided — edgeR differential testing in this module requires predefined phenotype groups.

## Inputs

- lipid feature matrix (lipid abundance/intensity table)
- phenotype file defining the contrast groups
- params.yml configuration for the MOI lipids module

## Outputs

- lipid_class_heatmap.png (ComplexHeatmap heatmap of lipid-class differential results)
- differential abundance tables for lipid classes, chain length, and saturation (text files)
- intermediate objects saved as RData files under the lipids module output directory

## How to apply

Load the lipid feature matrix and phenotype file specified in the params.yml of the MOI lipids module. Normalize lipid intensities with the mstus approach (built on preprocessCore preprocessing utilities), the normalization MOI applies specifically to proteins and lipids. Use lipidr to parse lipid identifiers into lipid-class, total chain length, and saturation features and to compute per-feature abundance summaries that associate genes with lipid classes present in the data via LipiDB's manually curated database. Run differential abundance analysis on the lipid features (classes, chain lengths, saturation states) with edgeR, contrasting the phenotype groups defined in the phenotype file. Generate the differential-results heatmap with ComplexHeatmap, saved as lipid_class_heatmap.png, and write differential tables and intermediate objects as text and RData files under the lipids module directory of the user-defined output folder.

## Related tools

- **lipidr** (parses lipid identifiers into lipid-class, chain-length and saturation features, annotates lipid classes, and computes per-feature abundance summaries; also drives enrichment of lipid classes, chain length, and degree of saturation)
- **edgeR** (performs differential abundance testing on lipid features across phenotype groups)
- **preprocessCore** (provides preprocessing utilities underlying the mstus normalization applied to protein and lipid intensities)
- **ComplexHeatmap** (renders the lipid-class heatmap of differential results (lipid_class_heatmap.png))
- **LipiDB** (annotates lipid species and associates genes with specific lipid classes present in the data using a manually curated database)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline hosting the lipids module, including filtering, normalization and differential analysis steps) — https://github.com/ASAGlab/moi

## Examples

```
nextflow run nf-core/mom -c nf-core/mom/params_lipids.yml -profile docker
```

## Evaluation signals

- lipid_class_heatmap.png exists in the lipids module directory of the user-defined output folder
- differential lipidomics tables are present in both text and RData format under the lipids module directory
- results cover all three lipid feature dimensions reported by the module: lipid class, chain length, and degree of saturation of the hydrocarbon chains
- lipid intensities were normalized with the mstus method (preprocessCore-based), matching MOI's protein/lipid normalization convention
- edgeR contrasts correspond exactly to the phenotype groups defined in the phenotype file

## Limitations

- The LipiDB gene–lipid-class association relies on a manually curated database, so coverage is limited to curated lipid classes present in that database.
- Missing-signal notes in the article flag that software versions for pipeline tools are not specified, complicating exact reproduction.
- Normalization with mstus is applied specifically to proteins and lipids; reusing it for other omics layers in MOI is not supported by the documented workflow.

## Evidence

- [intro] The pipeline performs lipidomics analysis: [evidence span withheld — anchor status: not_found]
- [readme] MOI pipeline scope statement: "multiOmicsIntegrator is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
- [readme] Reproducibility via containers: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
- [intro] Preprocessing hooks before differential analysis: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
