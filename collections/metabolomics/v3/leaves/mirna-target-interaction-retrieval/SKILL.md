---
name: mirna-target-interaction-retrieval
description: Use when integrating miRNA data with other omics (genes, isoforms, proteins,
  lipids) in MOI and you need to annotate putative miRNA-mRNA regulatory connections
  among the loaded feature matrices before downstream analysis.
when_to_use_negative:
- The input data contain no miRNA layer (e.g. mRNA-only RNA-seq or proteomics-only
  runs), since there are no miRNA features to query against targets
- You already possess a curated, validated miRNA-target annotation for your features
  and only need correlation or MCIA integration
- The goal is standalone pathway enrichment on a single omics dataset without integration
  or regulatory-edge annotation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: multiMiR
  role: R package used to retrieve putative miRNA-target interactions annotating miRNA-mRNA
    regulatory connections among the loaded features
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline whose integration module orchestrates the retrieval step
    and writes text/RData outputs
  repo: https://github.com/ASAGlab/moi
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:160ba0fa919184a5dace737c01ce2764b9037a5e3bb71f7e144173898430ae49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - BioTranslator
  - MultiOmicsIntegrator (MOI)
  - R package stats
  - clusterProfiler
  - multiMiR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# miRNA-target interaction retrieval

## Summary

Retrieves putative miRNA-mRNA regulatory interactions (via multiMiR) to annotate connections among features loaded into the MOI multi-omics integration module, prior to correlation-based filtering and pathway enrichment.

## When to use

Use when integrating miRNA data with other omics (genes, isoforms, proteins, lipids) in MOI and you need to annotate putative miRNA-mRNA regulatory connections among the loaded feature matrices before downstream analysis.

## When NOT to use

- The input data contain no miRNA layer (e.g. mRNA-only RNA-seq or proteomics-only runs), since there are no miRNA features to query against targets
- You already possess a curated, validated miRNA-target annotation for your features and only need correlation or MCIA integration
- The goal is standalone pathway enrichment on a single omics dataset without integration or regulatory-edge annotation

## Inputs

- Per-omics feature matrices (genes, miRNAs, isoforms, proteins, lipids) with aligned sample identifiers
- Phenotype file referenced by the integration module's params.yml
- miRNA and mRNA feature identifiers (to constrain the multiMiR query to features present in the data)

## Outputs

- Annotated miRNA-mRNA regulatory interaction table among loaded features
- Intermediate results written as text and RData objects into the hierarchical output structure (e.g. /user_defined_output_directory/<omics>/<tool>/)

## How to apply

Within the MOI integration module, load the per-omics feature matrices (genes, miRNAs, isoforms, proteins, lipids) plus the phenotype file using the input directories specified in the governing params.yml. Then call multiMiR to retrieve miRNA-target interactions restricted to the features actually present in the loaded matrices, annotating putative miRNA-mRNA regulatory connections. The rationale is that these retrieved connections define candidate edges that the subsequent Pearson or Spearman correlation analysis (R stats) will filter, 'aiding the noise mitigation for downstream analyses like pathway enrichment analysis'. Only interactions surviving correlation-based filtering are passed forward, so retrieval must cover the same feature identifiers used by the MCIA and correlation steps.

## Related tools

- **multiMiR** (R package used to retrieve putative miRNA-target interactions annotating miRNA-mRNA regulatory connections among the loaded features)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline whose integration module orchestrates the retrieval step and writes text/RData outputs) — https://github.com/ASAGlab/moi

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- The miRNA-target interaction annotations cover only features present in the loaded per-omics matrices (identifier overlap check)
- Retrieved interaction objects persist as text and RData files under the user-defined output directory
- Downstream Pearson/Spearman correlation filtering runs on the annotated candidate connections without identifier mismatches
- Samples still differentiate by phenotype in the MCIA sample-space PCA after the annotated connections feed the integration workflow
- The pipeline log shows the retrieval step executed within the integration module before pathway enrichment with clusterProfiler/BioTranslator

## Limitations

- Retrieved interactions are putative, not validated, and require correlation-based filtering with Pearson or Spearman to mitigate noise before pathway enrichment
- Feature identifier conventions between multiMiR and the per-omics matrices (genes, miRNAs, isoforms, proteins, lipids) must match or connections will be silently dropped
- The retrieval step is part of MOI's integration module and inherits its params.yml-driven input directory layout; running outside that structure loses the output RData/text conventions

## Evidence

- [other] Retrieve miRNA-target interactions with multiMiR to annotate putative miRNA-mRNA regulatory connections among the loaded features.: [evidence span withheld — anchor status: not_found]
- [intro] Correlation-based filtering rationale: "Correlation analysis performed with Pearson or Spearman can filter illuminating connection and patterns and aiding the noise mitigation for downstream analyses like pathway enrichment analysis"
- [intro] MOI covers miRNA-level RNA-seq analysis: [evidence span withheld — anchor status: not_found]
