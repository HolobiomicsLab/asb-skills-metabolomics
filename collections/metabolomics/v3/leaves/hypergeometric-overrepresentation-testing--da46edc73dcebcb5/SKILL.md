---
name: hypergeometric-overrepresentation-testing
description: Use when interpreting ICA metagenes (sk vectors) from cancer transcriptome/omics
  decompositions and you need to map top-contributing genes onto functional gene-set
  annotations.
when_to_use_negative:
- The input is not an ICA decomposition (no sk vectors / metagene gene weight rankings)
  — e.g., raw expression matrices not yet factorized.
- The goal is ICA computation or stability/component-number estimation itself, which
  is handled by BIODICA/stabilized-ica rather than post-hoc gene-set enrichment.
- The task is immune-cell deconvolution of transcriptomes, for which the review's
  companion tool DeconICA is the dedicated choice.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Jupyter Notebook
  role: Provides the example ICA ranked gene lists and analysis environment accompanying
    the review
  repo: github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python 3.7.3
  role: Runtime for the companion notebook and for scipy.stats.hypergeom overrepresentation
    testing
  repo: github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: scipy (scipy.stats.hypergeom)
  role: Hypergeometric overrepresentation test of metagene top-gene lists against
    GO/KEGG gene sets
- name: gseapy
  role: GSEA on the full sk-vector ranking to detect coordinated moderate shifts missed
    by overrepresentation testing
- name: BIODICA Navigator
  role: 'Alternative interface: toppgene analysis on an ICA metagene file via the
    BIODICAPipeLine command line'
  repo: LabBandSB/BIODICA
- name: pip
  role: Installs the notebook dependencies listed in Jupyter_notebook_package_requirements.txt
  repo: github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:fa96e8873b49a364cdb6687ea616671ff424f509cca35e812d0a57ad08de6d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIODICA Navigator
  - Jupyter Notebook
  - Python
  - Python 3.7.3
  - gseapy
  - pip
  - scipy (scipy.stats.hypergeom)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Hypergeometric overrepresentation testing of ICA metagene contributions

## Summary

Assigns biological meaning to independent component (IC) sk vectors from cancer omics data by testing whether the most-contributing genes of each metagene are enriched for curated gene sets (GO/KEGG), complemented by GSEA on the full sk-vector ranking.

## When to use

Use when interpreting ICA metagenes (sk vectors) from cancer transcriptome/omics decompositions and you need to map top-contributing genes onto functional gene-set annotations.

## When NOT to use

- The input is not an ICA decomposition (no sk vectors / metagene gene weight rankings) — e.g., raw expression matrices not yet factorized.
- The goal is ICA computation or stability/component-number estimation itself, which is handled by BIODICA/stabilized-ica rather than post-hoc gene-set enrichment.
- The task is immune-cell deconvolution of transcriptomes, for which the review's companion tool DeconICA is the dedicated choice.

## Inputs

- Ranked gene list (sk vector) for each ICA metagene from the companion Jupyter notebook
- Per-metagene top-contributing gene lists selected by the heavy-tail heuristic (Section 2.6)
- Curated gene-set library (GO/KEGG terms)
- Expressed-gene universe used as hypergeometric background

## Outputs

- enrichment_results.csv combining gene-set names, hypergeometric p-values, and Benjamini-Hochberg FDR values per metagene
- GSEA enrichment results on the full ranked metagene gene lists

## How to apply

Load the ranked gene lists for each ICA metagene from the companion Jupyter notebook materials (Python 3.7.3, dependencies installed via pip from Jupyter_notebook_package_requirements.txt, repo github.com/sysbio-curie/ICA-in-Cancer-research-review-materials). Select the most-contributing genes per metagene with the heavy-tail heuristic on gene weights described in the review's interpretation section (Section 2.6). Run a hypergeometric overrepresentation test of each top-gene list against a curated gene-set library (e.g., GO/KEGG) using scipy.stats.hypergeom, with the expressed-gene universe as the background. In parallel, run GSEA (e.g., via gseapy) on the full ranked sk vector to capture coordinated moderate shifts, and apply Benjamini-Hochberg FDR correction across gene sets. The BIODICA Navigator offers an equivalent toppgene analysis on a metagene file via its command-line pipeline.

## Related tools

- **Jupyter Notebook** (Provides the example ICA ranked gene lists and analysis environment accompanying the review) — github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python 3.7.3** (Runtime for the companion notebook and for scipy.stats.hypergeom overrepresentation testing) — github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **scipy (scipy.stats.hypergeom)** (Hypergeometric overrepresentation test of metagene top-gene lists against GO/KEGG gene sets)
- **gseapy** (GSEA on the full sk-vector ranking to detect coordinated moderate shifts missed by overrepresentation testing)
- **BIODICA Navigator** (Alternative interface: toppgene analysis on an ICA metagene file via the BIODICAPipeLine command line) — LabBandSB/BIODICA
- **pip** (Installs the notebook dependencies listed in Jupyter_notebook_package_requirements.txt) — github.com/sysbio-curie/ICA-in-Cancer-research-review-materials

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene
```

## Evaluation signals

- enrichment_results.csv exists with the expected columns: gene-set names, p-values, and FDR values for each metagene.
- All reported FDR values are Benjamini-Hochberg corrected and fall within [0, 1]; p-values within [0, 1].
- The hypergeometric background equals the expressed-gene universe, not the whole genome, and the top-gene count follows the heavy-tail heuristic rather than an arbitrary fixed cutoff.
- GSEA results on the full ranked list are consistent with (and can complement) the overrepresentation hits for the same metagene.
- Metagene identifiers and gene-set hits are reproducible from the notebook's ranked gene lists.

## Limitations

- Overrepresentation testing depends on the choice of top-gene set (heavy-tail heuristic) and background universe; different choices change which gene sets are called significant.
- The hypergeometric test detects only strong shifts among top genes; coordinated moderate shifts require the complementary GSEA on the full sk vector.
- Multiple-testing burden across gene sets requires FDR correction; uncorrected p-values will over-call enrichments.

## Evidence

- [abstract] Companion notebook availability: "This review is accompanied by interactive Jupyter notebook located at https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials."
- [readme] BIODICA toppgene analysis on metagene files: [evidence span withheld — anchor status: external_doc]
- [readme] BIODICA command for gene-set enrichment on ICA metagenes: [evidence span withheld — anchor status: external_doc]
- [readme] BIODICA interpretation analyses: [evidence span withheld — anchor status: external_doc]
