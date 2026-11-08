---
name: fdr-multiple-testing-correction
description: Use when many gene sets (e.g., GO/KEGG terms) have been tested per ICA
  metagene Sk vector via hypergeometric overrepresentation or GSEA and p-values must
  be corrected before calling enriched sets.
when_to_use_negative:
- Only one gene set or one hypothesis is being tested — no family-wise or FDR adjustment
  is needed
- The p-values come from different metagenes but the analysis goal is component-wise
  exploration rather than a single corrected call list; correcting across all components
  and all gene sets at once may be overly conservative
- The input is a ranked gene list that has not yet been subjected to any enrichment
  test — FDR correction applies to test p-values, not to rankings directly
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Jupyter notebook
  role: Companion Python notebook materials from which the ranked gene lists per ICA
    metagene are loaded and where hypergeometric testing and Benjamini-Hochberg FDR
    correction are executed
  repo: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- name: BIODICA
  role: GUI/command-line environment whose toppgene analysis performs functional interpretation
    (including corrected enrichment) on metagene files from earlier ICA decompositions,
    without recomputing ICA
  repo: https://github.com/LabBandSB/BIODICA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/ijms20184414
    title: Independent Component Analysis for Unraveling the Complexity of Cancer
      Omics Datasets
  source_content_hash: sha256:fc5f41b972854897c1629e28362ab5080f1ca1695d7624b16972037e55b5819f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
  tools:
  - BIODICA
  - Jupyter notebook
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# fdr-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg false discovery rate (FDR) correction to the p-values produced by functional enrichment testing (hypergeometric overrepresentation and GSEA) of ICA metagene (Sk) gene lists, so that enriched gene sets are called at a controlled false discovery rate across many simultaneous gene-set tests.

## When to use

Use when many gene sets (e.g., GO/KEGG terms) have been tested per ICA metagene Sk vector via hypergeometric overrepresentation or GSEA and p-values must be corrected before calling enriched sets.

## When NOT to use

- Only one gene set or one hypothesis is being tested — no family-wise or FDR adjustment is needed
- The p-values come from different metagenes but the analysis goal is component-wise exploration rather than a single corrected call list; correcting across all components and all gene sets at once may be overly conservative
- The input is a ranked gene list that has not yet been subjected to any enrichment test — FDR correction applies to test p-values, not to rankings directly

## Inputs

- Vector of raw p-values from hypergeometric overrepresentation tests of top-contributing genes against a curated gene-set library (GO/KEGG terms)
- Vector of raw p-values (or NES-based p-values) from GSEA run on full Sk-vector ranked gene lists
- Gene-set names and universe definition (expressed-gene background) for the tested enrichment results

## Outputs

- Benjamini-Hochberg adjusted q-values (FDR) per gene set per metagene
- Combined enrichment results table (enrichment_results.csv) with gene-set names, raw p-values, and FDR values

## How to apply

After running the two complementary enrichment strategies on each ICA metagene — a hypergeometric overrepresentation test (scipy.stats.hypergeom) on the most-contributing genes selected by the heavy-tail heuristic (Section 2.6), and GSEA (e.g., gseapy) on the full Sk-vector gene ranking — collect all gene-set p-values into a single results table. Correct across the family of gene-set tests using the Benjamini-Hochberg FDR procedure so that the expected proportion of false positives among called gene sets is bounded. Attach the corrected q-values alongside raw p-values and gene-set names in the combined table. Judge correctness by confirming the FDR column is monotone in the p-value column and that the number of significant gene sets shrinks (or stays equal) relative to raw p < 0.05 calls. The BIODICA environment exposes a similar functional-interpretation step via its toppgene analysis, which can be run on a precomputed metagene file without recomputing the ICA decomposition.

## Related tools

- **Jupyter notebook** (Companion Python notebook materials from which the ranked gene lists per ICA metagene are loaded and where hypergeometric testing and Benjamini-Hochberg FDR correction are executed) — https://github.com/sysbio-curie/ICA-in-Cancer-research-review-materials
- **BIODICA** (GUI/command-line environment whose toppgene analysis performs functional interpretation (including corrected enrichment) on metagene files from earlier ICA decompositions, without recomputing ICA) — https://github.com/LabBandSB/BIODICA

## Examples

```
java -cp BIODICA_GUI.jar BIODICAPipeLine -config config -sfile [path_to_metagene_file] -dotoppgene
```

## Evaluation signals

- FDR q-values are monotonically non-decreasing with raw p-values within each correction family
- Every gene set with q < 0.05 also has raw p < 0.05, and the significant set under FDR is a subset of the raw-p significant set
- The corrected table contains all three required columns (gene-set names, p-values, FDR values) for every tested metagene
- Number of FDR-significant gene sets per metagene is smaller than or equal to the number passing uncorrected p < 0.05
- The background universe used by scipy.stats.hypergeom matches the expressed-gene universe, so hypergeometric p-values feeding the correction are well-defined

## Limitations

- The review pairs overrepresentation testing with GSEA precisely because the hypergeometric test on the most-contributing genes misses coordinated moderate shifts across the whole Sk-vector ranking; FDR correction on the overrepresentation p-values alone does not recover those signals
- The choice of background universe (expressed genes) strongly affects hypergeometric p-values and therefore the corrected FDR calls
- BIODICA's MATLAB ICA core is described as being of interest 'only for historical and compatibility reasons'; enrichment interpretation is run on precomputed metagene files rather than recomputed decompositions

## Evidence

- [full_text] Standard enrichment workflow on Sk vectors: [evidence span withheld — anchor status: not_found]
- [full_text] Benjamini-Hochberg correction step: [evidence span withheld — anchor status: not_found]
- [readme] BIODICA toppgene analysis on precomputed metagene file: [evidence span withheld — anchor status: external_doc]
- [abstract] Companion notebook as practical ICA tool: "Our review is accompanied by a Jupyter notebook which illustrates the discussed concepts and provides a practical tool for applying ICA to the analysis of cancer omics datasets."
