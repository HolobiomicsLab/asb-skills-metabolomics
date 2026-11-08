---
name: gene-set-enrichment-analysis
description: 'Use when interpreting ICA metagenes (sk vectors) from cancer omics data:
  you have a ranked gene list per component and need GO/KEGG functional enrichment
  to name the biological signal.'
when_to_use_negative:
- Input is a raw expression matrix rather than an ICA-derived sk vector — the ICA
  decomposition (e.g., via the BIODICA Navigator or the companion notebook) must be
  computed first
- Only a short unranked gene list is available without its full sk-vector ranking
  — GSEA on the ranking cannot be run, only the overrepresentation branch applies
- The goal is ICA component number selection, stability analysis, or deconvolution
  (BIODICA stability estimation or DeconICA) rather than functional interpretation
  of components
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Jupyter Notebook
  role: Python 3.7.3 notebook accompanying the review that provides the ranked metagene
    gene lists used as enrichment input
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: gseapy
  role: Python implementation for running GSEA on the full ranked sk-vector gene lists
- name: BIODICA Navigator
  role: GUI/command-line environment whose toppgene analysis can be launched on a
    computed metagene file for overrepresentation interpretation of ICA results
  repo: https://github.com/LabBandSB/BIODICA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:d422566f39d250dcab61cb2b1463ed300f8e4853d865b564b31809fa53825b82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA Navigator
  - Jupyter Notebook
  - Python
  - gseapy
  - pip
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-set-enrichment-analysis

## Summary

Assigns biological meaning to ICA metagenes (sk vectors) from cancer omics data by combining hypergeometric overrepresentation testing of the most-contributing genes with Gene Set Enrichment Analysis on the full sk-vector ranking, against curated GO/KEGG gene-set libraries.

## When to use

Use when interpreting ICA metagenes (sk vectors) from cancer omics data: you have a ranked gene list per component and need GO/KEGG functional enrichment to name the biological signal.

## When NOT to use

- Input is a raw expression matrix rather than an ICA-derived sk vector — the ICA decomposition (e.g., via the BIODICA Navigator or the companion notebook) must be computed first
- Only a short unranked gene list is available without its full sk-vector ranking — GSEA on the ranking cannot be run, only the overrepresentation branch applies
- The goal is ICA component number selection, stability analysis, or deconvolution (BIODICA stability estimation or DeconICA) rather than functional interpretation of components

## Inputs

- Ranked gene lists (sk vectors) for each ICA metagene from the companion Jupyter notebook materials
- Top-contributing gene lists per metagene selected by the heavy-tail heuristic on sk-vector weights
- Curated gene-set library (e.g., GO or KEGG terms)
- Expressed-gene universe used as hypergeometric background

## Outputs

- enrichment_results.csv combining gene-set names, p-values, and Benjamini-Hochberg FDR values per ICA metagene
- Per-metagene overrepresentation test results (hypergeometric p-values)

## How to apply

Load the ranked gene lists for each ICA metagene from the companion Jupyter notebook materials (github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, Python 3.7.3, pip-installable requirements). Select the most-contributing genes per metagene with the heavy-tail heuristic on gene weights described in the review's interpretation section (Section 2.6). Run a hypergeometric overrepresentation test of each top-gene list against a curated gene-set library (GO/KEGG terms) using scipy.stats.hypergeom with the expressed-gene universe as background. In parallel, run GSEA (e.g., gseapy) on the full sk-vector ranking to detect coordinated moderate gene shifts the overrepresentation test misses. Correct for multiple testing across gene sets with Benjamini-Hochberg FDR, then assemble gene-set names, p-values, and FDR values into enrichment_results.csv.

## Related tools

- **Jupyter Notebook** (Python 3.7.3 notebook accompanying the review that provides the ranked metagene gene lists used as enrichment input) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **gseapy** (Python implementation for running GSEA on the full ranked sk-vector gene lists)
- **BIODICA Navigator** (GUI/command-line environment whose toppgene analysis can be launched on a computed metagene file for overrepresentation interpretation of ICA results) — https://github.com/LabBandSB/BIODICA

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene
```

## Evaluation signals

- enrichment_results.csv contains the expected columns: gene-set names, p-values, and FDR values
- All FDR values lie in [0, 1] and result from Benjamini-Hochberg correction applied across gene sets
- Hypergeometric p-values are consistent with the chosen expressed-gene universe size (larger universe raises background count and changes significance)
- GSEA on the full sk ranking recovers gene sets with only moderate coordinated shifts that the top-gene overrepresentation test fails to flag
- Re-running the notebook materials from the review repository reproduces the same ranked gene lists for each ICA metagene

## Limitations

- Overrepresentation testing on the top-contributing genes alone misses coordinated moderate shifts, which is why the full-ranking GSEA branch is required
- The companion notebook environment is pinned to Python 3.7.3 with dependencies from Jupyter_notebook_package_requirements.txt
- The BIODICA MATLAB computational core is described as 'of interest only for historical and compatibility reasons'; the Python core (stabilized-ica) is the default

## Evidence

- [methods] Run hypergeometric overrepresentation testing of each metagene's top-gene list against a curated gene-set library (e.g., GO/KEGG terms): [evidence span withheld — anchor status: not_found]
- [methods] Run Gene Set Enrichment Analysis (GSEA, e.g., via gseapy) on the full ranked metagene gene lists: [evidence span withheld — anchor status: not_found]
- [abstract] Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
- [readme] the toppgene analysis can be launched by executing java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene: "the toppgene analysis can be launched by executing
``njava -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene
"
