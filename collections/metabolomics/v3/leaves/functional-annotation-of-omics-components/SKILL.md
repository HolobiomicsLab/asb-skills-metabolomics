---
name: functional-annotation-of-omics-components
description: Use when you have an ICA-derived metagene (sk vector) of ranked gene
  weights from cancer omics data and need to identify enriched biological terms or
  pathways to interpret the component.
when_to_use_negative:
- The decomposition output is a sample-vector (aS) mixing matrix rather than a gene-weight
  sk vector — annotation applies to gene loadings.
- The goal is to compute or stabilize the ICA decomposition itself (choose number
  of components, run fastICA) rather than interpret an already-computed metagene.
- There is no ranked gene list available — the method requires the whole sk-vector
  ranking for GSEA and a top-gene subset for hypergeometric testing.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: Jupyter notebook
  role: Supplies the ranked ICA metagene gene lists and executable example accompanying
    the review
  repo: github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: Python
  role: Python 3.7.3 environment in which hypergeometric testing (scipy.stats.hypergeom)
    and GSEA via gseapy are run
- name: pip
  role: Installs notebook dependencies from Jupyter_notebook_package_requirements.txt
- name: BIODICA Navigator
  role: Can perform downstream interpretation analyses of computed ICA results, including
    top-gene enrichment, without recomputing the decomposition
  repo: https://sysbio-curie.github.io/biodica-environment/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:35dbbc6ba191f7172b45853e1b5eab9c724037563b34bfd7e04c05e20f5b4cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BIODICA Navigator
  - Jupyter notebook
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# functional-annotation-of-omics-components

## Summary

Assigns biological meaning to ICA metagenes (sk vectors) from cancer omics data by combining hypergeometric overrepresentation testing of the most-contributing genes with Gene Set Enrichment Analysis (GSEA) on the full sk-vector gene ranking, the workflow the review identifies as standard practice for metagene interpretation.

## When to use

Use when you have an ICA-derived metagene (sk vector) of ranked gene weights from cancer omics data and need to identify enriched biological terms or pathways to interpret the component.

## When NOT to use

- The decomposition output is a sample-vector (aS) mixing matrix rather than a gene-weight sk vector — annotation applies to gene loadings.
- The goal is to compute or stabilize the ICA decomposition itself (choose number of components, run fastICA) rather than interpret an already-computed metagene.
- There is no ranked gene list available — the method requires the whole sk-vector ranking for GSEA and a top-gene subset for hypergeometric testing.

## Inputs

- Ranked gene lists (sk vectors) for each ICA metagene from the companion Jupyter notebook
- Curated gene-set library (GO/KEGG terms)
- Expressed-gene universe used as hypergeometric background

## Outputs

- enrichment_results.csv combining gene-set names, p-values, and Benjamini-Hochberg FDR values
- Per-metagene overrepresentation results (most-contributing gene sets)
- Per-metagene GSEA results on full ranked lists

## How to apply

Load the ranked gene lists for each ICA metagene from the companion Jupyter notebook materials (github.com/sysbio-curie/ICA-in-Cancer-research-review-materials, Python 3.7.3). Select the most-contributing genes per metagene using the heavy-tail heuristic on gene weights described in the review's interpretation section (Section 2.6). Run a hypergeometric overrepresentation test on each metagene's top-gene list against a curated gene-set library (GO/KEGG) with scipy.stats.hypergeom, using the expressed-gene universe as background. In parallel, run GSEA (e.g., via gseapy) on the full ranked metagene gene lists to detect coordinated moderate shifts not captured by the overrepresentation test. Correct for multiple testing across gene sets with Benjamini-Hochberg FDR and assemble a results table with gene-set names, p-values, and FDR values.

## Related tools

- **Jupyter notebook** (Supplies the ranked ICA metagene gene lists and executable example accompanying the review) — github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **Python** (Python 3.7.3 environment in which hypergeometric testing (scipy.stats.hypergeom) and GSEA via gseapy are run)
- **pip** (Installs notebook dependencies from Jupyter_notebook_package_requirements.txt)
- **BIODICA Navigator** (Can perform downstream interpretation analyses of computed ICA results, including top-gene enrichment, without recomputing the decomposition) — https://sysbio-curie.github.io/biodica-environment/

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile path/to/metagene_file -dotoppgene
```

## Evaluation signals

- Output table enrichment_results.csv exists with columns for gene-set names, p-values, and FDR values for every analyzed metagene.
- Hypergeometric background equals the expressed-gene universe, not the whole genome — check the universe size recorded in the analysis.
- All reported gene-set p-values carry a corresponding Benjamini-Hochberg FDR correction computed across the gene-set library.
- Top-gene lists derive from the heavy-tail heuristic on sk-vector weights (Section 2.6), not an arbitrary fixed cutoff.
- GSEA results show enrichment signals consistent with the overrepresentation test for strongly enriched terms, while additionally capturing moderate coordinated shifts.

## Limitations

- The overrepresentation test on most-contributing genes misses coordinated moderate shifts captured only by GSEA on the full ranking — the two tests are complementary and both are needed.
- Results depend on the heavy-tail heuristic for defining the most-contributing genes; a poor choice changes the hypergeometric input set.
- The MATLAB ICA core in BIODICA is maintained only for historical and compatibility reasons; the Python stabilized-ica core is the default for computing decompositions feeding this analysis.

## Evidence

- [results] The standard interpretation workflow applies functional enrichment to sk vectors: [evidence span withheld — anchor status: not_found]
- [abstract] Jupyter notebook location: "This review is accompanied by interactive Jupyter notebook located at https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials."
- [intro] Python version for notebook: "This notebook has been done with Python 3.7.3 (v3.7.3:ef4ec6ed12)."
- [readme] BIODICA enrichment without recomputation: "For example, the toppgene analysis can be launched by executing
``njava -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene"
- [intro] Dependencies installation: "All dependencies can be installed using:
pip install Jupyter_notebook_package_requirements.txt"
