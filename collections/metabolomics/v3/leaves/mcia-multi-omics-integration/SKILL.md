---
name: mcia-multi-omics-integration
description: Use when two or more omics feature matrices (e.g. genes, proteins, lipids/metabolomics)
  share a sample set with a phenotype of interest and the goal is data-driven cross-omics
  integration to differentiate samples by phenotype.
when_to_use_negative:
- The datasets have no shared samples or phenotype, so the matrices cannot be co-inertia-aligned
  on a common sample space
- Only a single omics layer is available — MOI's individual omics modules (e.g. DE
  with edgeR/DESeq2/RankProduct) are the appropriate path, not MCIA integration
- A hypothesis-driven supervised classifier is required rather than the unsupervised,
  data-driven projection MCIA provides
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Pipeline whose integration module runs MCIA and organizes the hierarchical
    outputs
  repo: https://github.com/ASAGlab/moi
- name: Nextflow
  role: Workflow engine (v23.04.2.5870) executing the MOI pipeline with Docker/Singularity
    profiles
  repo: https://www.nextflow.io
- name: clusterProfiler
  role: Pathway enrichment analysis performed on features after the MCIA integration
    step
- name: BioTranslator
  role: Alternative pathway enrichment tool applied to integrated results
- name: multiMiR
  role: Retrieves miRNA-target interactions to annotate miRNA-mRNA regulatory connections
    among integrated features
- name: R package stats
  role: Computes pairwise Pearson or Spearman correlations between omics feature matrices
    for noise filtering
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:0ea1d45b04c53899610fe397704ec0be906e015c90882f0d0819000ca9c9efd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - BioTranslator
  - Docker
  - FASTQC
  - ISOformSwitchAnalyzeR
  - MCIA (usage_mcia integration module)
  - MultiOmicsIntegrator (MOI)
  - Nextflow
  - R package stats
  - Singularity
  - Trim Galore
  - clusterProfiler
  - multiMiR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mcia-multi-omics-integration

## Summary

Apply Multiple Co-Inertia Analysis (MCIA) within the MultiOmicsIntegrator (MOI) Nextflow pipeline to project several omics layers (genes, miRNAs, isoforms, proteins, lipids) onto a common sample space, then feed the integrated features into correlation filtering and pathway enrichment.

## When to use

Use when two or more omics feature matrices (e.g. genes, proteins, lipids/metabolomics) share a sample set with a phenotype of interest and the goal is data-driven cross-omics integration to differentiate samples by phenotype.

## When NOT to use

- The datasets have no shared samples or phenotype, so the matrices cannot be co-inertia-aligned on a common sample space
- Only a single omics layer is available — MOI's individual omics modules (e.g. DE with edgeR/DESeq2/RankProduct) are the appropriate path, not MCIA integration
- A hypothesis-driven supervised classifier is required rather than the unsupervised, data-driven projection MCIA provides

## Inputs

- Per-omics count/feature matrices (genes, miRNAs, isoforms, proteins, lipids) aligned on a shared sample set
- Phenotype file defining the phenotype of interest (e.g. TNBC vs other BC subtypes)
- params.yml configuration for the integration module specifying input/output directories

## Outputs

- PCA of the MCIA sample space showing sample separation by phenotype with variables projected onto it
- Elbow plots for significant principal components and pseudo-eigen values per dataset
- Correlation analysis results (Pearson or Spearman) used for noise filtering
- Pathway enrichment results from clusterProfiler or BioTranslator on the integrated features
- Text tables, RData objects, and plots written under /user_defined_output_directory/<omics>/<tool>/

## How to apply

Load the per-omics feature matrices (genes, miRNAs, isoforms, proteins, lipids) together with the phenotype file, using the input directories specified in the params.yml file governing the integration module; optionally retrieve miRNA-target interactions with multiMiR to annotate miRNA-mRNA regulatory connections. Run MCIA to project all omics onto a common sample space and inspect the reported PCA of the sample space to check that samples differentiate according to the phenotype of interest, projecting variables onto that space to explore their contribution. Use elbow plots for significant principal components and pseudo-eigen values as an indication of each dataset's relative contribution to the observed variance. Compute pairwise Pearson or Spearman correlations between the omics feature matrices (R stats package) to filter noisy connections and patterns before downstream analyses, then pass the filtered, correlated features to pathway enrichment with clusterProfiler (which can be utilized after the integration step of MCIA) or BioTranslator. Write module outputs — informative plots plus intermediate results as text and RData objects — into the hierarchical output structure (e.g. /user_defined_output_directory/<omics>/<tool>/).

## Related tools

- **MultiOmicsIntegrator (MOI)** (Pipeline whose integration module runs MCIA and organizes the hierarchical outputs) — https://github.com/ASAGlab/moi
- **Nextflow** (Workflow engine (v23.04.2.5870) executing the MOI pipeline with Docker/Singularity profiles) — https://www.nextflow.io
- **clusterProfiler** (Pathway enrichment analysis performed on features after the MCIA integration step)
- **BioTranslator** (Alternative pathway enrichment tool applied to integrated results)
- **multiMiR** (Retrieves miRNA-target interactions to annotate miRNA-mRNA regulatory connections among integrated features)
- **R package stats** (Computes pairwise Pearson or Spearman correlations between omics feature matrices for noise filtering)

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- The MCIA PCA of the sample space exists and samples visibly differentiate according to the phenotype of interest (e.g. TNBC vs other subtypes)
- Variables are projected onto the sample space and their contributions can be inspected
- Elbow plots are present and pseudo-eigen values report each dataset's relative contribution to the observed variance
- Text tables and RData objects for the correlation and enrichment steps exist under /user_defined_output_directory/<omics>/<tool>/ (e.g. /user_defined_output_directory/genes/biotranslator/)
- In the CLLE TNBC use case, prioritized features XBP1 and DGAT2 appear among the differential/integrated results consistent with the published Results

## Limitations

- MCIA output interpretation relies on subjective inspection of the PCA sample separation and elbow plots to select significant components
- Correlation-based filtering (Pearson/Spearman) removes noisy connections but also risks discarding true weak signals before enrichment
- MOI documentation does not specify software versions for all modules (listed as a missing signal), and the release notes contain empty 'Added'/'Fixed'/'Dependencies' sections

## Evidence

- [intro] MCIA PCA separates samples by phenotype: "MCIA reports the PCA of the sample space where we can see how samples differentiate according to the phenotype of interest."
- [intro] Enrichment runs after MCIA integration: "Clusterprofiler can be utilized by individual analyses or after the integration step of MCIA."
- [intro] Correlation filtering rationale: "Correlation analysis performed with Pearson or Spearman can filter illuminating connection and patterns and aiding the noise mitigation for downstream analyses like pathway enrichment analysis"
- [readme] Pipeline scope includes integration: "multiOmicsIntegrator is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
