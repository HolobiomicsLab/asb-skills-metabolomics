---
name: gene-set-enrichment-analysis
description: Use when ionomics data (e.g. ICP-MS ion concentrations for haploid S.
  cerevisiae deletion mutants) have been clustered and you need to identify statistically
  significant clusters and associated biological processes via GO Slim annotation
  and GO terms enrichment.
when_to_use_negative:
- The input has not passed through IonFlow Pre-processing (raw, unbatch-corrected
  concentration data) — the enrichment section requires the cleaned dataset produced
  by pre-processing first
- The goal is element-element correlation or relevance network extraction rather than
  biological-process over-representation of mutant clusters
- The dataset is not from an organism with available GO/GO Slim annotations compatible
  with the IonFlow package's bundled resources
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: IonFlow
  role: R package / Galaxy tool whose clustering section performs GO Slim annotation
    and GO terms enrichment on ion-profile clusters
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Platform hosting the IonFlow enrichment tool with a user-friendly interface
  repo: https://github.com/galaxyproject/galaxy
- name: MetaboFlow (IonFlow R package)
  role: Underlying R package containing the four-section workflow (Pre-processing,
    Exploratory analysis, Clustering incl. GO Slim annotation and GO terms enrichment,
    Network analysis)
  repo: https://github.com/AlinaPeluso/MetaboFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:9ed8734b193169dedb8dd3c514be3d279b5617da75b5e75c89ba8c4100001ad8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - MetaboFlow (IonFlow R package)
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-set-enrichment-analysis

## Summary

Statistically identify over-represented biological processes and Gene Ontology (GO Slim) annotations among clusters of ion-profile-similar yeast deletion mutants, as implemented in the IonFlow R/Galaxy pipeline for ICP-MS ionomics data.

## When to use

Use when ionomics data (e.g. ICP-MS ion concentrations for haploid S. cerevisiae deletion mutants) have been clustered and you need to identify statistically significant clusters and associated biological processes via GO Slim annotation and GO terms enrichment.

## When NOT to use

- The input has not passed through IonFlow Pre-processing (raw, unbatch-corrected concentration data) — the enrichment section requires the cleaned dataset produced by pre-processing first
- The goal is element-element correlation or relevance network extraction rather than biological-process over-representation of mutant clusters
- The dataset is not from an organism with available GO/GO Slim annotations compatible with the IonFlow package's bundled resources

## Inputs

- Cleaned/standardised ionome dataset output by IonFlow Pre-processing (OD-corrected ICP-MS ion concentrations in ppm, e.g. the haploid S. cerevisiae deletion-mutant ionome)
- Cluster assignment of mutants from IonFlow hierarchical clustering
- Gene ontology / GO Slim annotation resources for the organism (bundled with the IonFlow R package)

## Outputs

- Statistically significant clusters with associated GO terms and biological-process enrichments
- GO Slim annotations of mutant clusters
- Biologically interpretable cluster labels supporting validation of findings such as SMF2 impact scores, MTF1 network betweenness, and mitochondrial enrichment

## How to apply

In the IonFlow pipeline (an R package wrapped as a Galaxy tool), gene set enrichment analysis runs as the fourth section, after Pre-processing (plate-median batch correction on log-transformed concentrations, 3-std outer-fence outlier removal, standardisation) has produced the cleaned dataset. Clustering of mutants by ion-profile-specific similarity measures feeds the enrichment step, which assigns GO Slim annotations to clusters and tests GO term over-representation to identify statistically significant clusters and associated biological processes. For the S. cerevisiae ionome benchmark (14 ions: Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66 measured in ppm, OD-corrected, 2/4/8 replicates per knockout with YDL227C as control), significant clusters are then interpreted biologically — e.g. validating SMF2 impact scores, MTF1 network betweenness, and mitochondrial enrichment against the iHUB case-study findings. There is no required ordering of clustering, network analysis, and enrichment beyond running pre-processing first.

## Related tools

- **IonFlow** (R package / Galaxy tool whose clustering section performs GO Slim annotation and GO terms enrichment on ion-profile clusters) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Platform hosting the IonFlow enrichment tool with a user-friendly interface) — https://github.com/galaxyproject/galaxy
- **MetaboFlow (IonFlow R package)** (Underlying R package containing the four-section workflow (Pre-processing, Exploratory analysis, Clustering incl. GO Slim annotation and GO terms enrichment, Network analysis)) — https://github.com/AlinaPeluso/MetaboFlow

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- Enrichment runs only on the output of Pre-processing — verify the cleaned dataset (median-batch-corrected, outlier-filtered, standardised) precedes the enrichment call
- Statistically significant clusters must be reported with their associated biological processes; check that GO terms/GO Slim annotations accompany each significant cluster
- For the iHUB yeast case study, results should be consistent with the predefined findings: ~75% (3171/4945) unchanged symbolic profiles, exponential decrease of probability with number k of altered ions, high Mg–P correlation, SMF2 impact scores, MTF1 betweenness, and mitochondrial enrichment
- Cross-check that the clustered mutants were grouped using ion-profile-specific similarity measures rather than generic distance metrics
- Confirm the ion set matches the expected 14-element ICP-MS panel (Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn) before interpreting enrichments

## Limitations

- The article reports no quantitative benchmark results or performance metrics for the two case studies, so enrichment significance must be validated against external predefined findings
- Supplementary material is referenced only by file size (PDF 1754 kb) without description of contents, limiting reproducibility of enrichment parameters
- No version number stated for IonFlow or Galaxy, complicating exact reproduction of enrichment results
- Enrichment applicability was demonstrated on yeast and HeLa ionome benchmarks; extension to other organisms depends on available GO annotations

## Evidence

- [abstract] The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome.: "The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome."
- [abstract] including principal components analysis, hierarchical clustering, relevance network extraction and analysis, and gene set enrichment analysis: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis, and gene set enrichment analysis"
- [readme] Clustering which also includes the GO Slim annotation and the GO terms enrichment: [evidence span withheld — anchor status: external_doc]
- [readme] The pre-processing section is required first as it produces in output the cleaned dataset to be used in the other sections.: [evidence span withheld — anchor status: external_doc]
- [readme] ICP-MS data of yeast intracellular ion concentrations measured for 1454 single-gene haploid knockouts: [evidence span withheld — anchor status: external_doc]
- [discussion] ion-profile-specific similarity measures (Iacovacci et al., 2020): "ion-profile-specific similarity measures (Iacovacci et al., 2020) that optionally, can be used by the user"
