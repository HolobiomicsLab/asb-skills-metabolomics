---
name: complementary-method-comparison-visualization
description: Use when two annotation methods have been run over the same query spectra
  and each query carries a Boolean correctness label per method, and the goal is to
  quantify and visualize overlap of correct annotations (contingency table / Venn-style
  breakdown).
when_to_use_negative:
- Only one method's results are available, so no paired per-query correctness labels
  exist to cross-tabulate.
- The goal is absolute accuracy benchmarking of a single method rather than overlap/complementarity
  between two methods on a shared query set.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BAM
  role: Method under evaluation; its per-query correctness labels come from the Zenodo
    deposit
  repo: https://github.com/HassounLab/BAM
- name: SIRIUS 5.6.3
  role: Baseline method applied to the same 60,368 anchor–suspect pair query spectra
    to produce comparison correctness labels
- name: pandas / scikit-learn 1.1.2
  role: Build the correctness contingency table and compute both-correct / method-unique
    counts
- name: Matplotlib / Seaborn
  role: Render the bar or Venn-style uniqueness breakdown of complementary correct-annotation
    counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:46583e2d6f449a9df657c347c3341148b9b1fffc1e796db7ab08ccd9487d259d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - Matplotlib
  - Matplotlib / Seaborn
  - NumPy
  - PROXIMAL2
  - Pandas
  - Python 3.9
  - SIRIUS 5.6.3
  - Seaborn
  - pandas / scikit-learn 1.1.2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# complementary-method-comparison-visualization

## Summary

Builds a per-query correctness contingency table across two annotation methods (BAM vs SIRIUS 5.6.3) on the same 60,368 anchor–suspect pair query set, and visualizes the overlap (both-correct, BAM-only, SIRIUS-only) to test whether the methods are complementary rather than redundant.

## When to use

Use when two annotation methods have been run over the same query spectra and each query carries a Boolean correctness label per method, and the goal is to quantify and visualize overlap of correct annotations (contingency table / Venn-style breakdown).

## When NOT to use

- Only one method's results are available, so no paired per-query correctness labels exist to cross-tabulate.
- The goal is absolute accuracy benchmarking of a single method rather than overlap/complementarity between two methods on a shared query set.

## Inputs

- BAM evaluation results (correctness labels per anchor–suspect query) from Zenodo DOI 10.5281/zenodo.13903946
- SIRIUS 5.6.3 baseline evaluation results with per-query correctness labels on the same 60,368 anchor–suspect pair set
- Shared query identifier list (anchor–suspect pairs from the global molecular network)

## Outputs

- 2x2 contingency table of BAM correctness vs SIRIUS correctness (CSV)
- Uniqueness breakdown figure (both-correct, BAM-only, SIRIUS-only counts) as PNG
- Overlap counts supporting (or refuting) the complementarity claim

## How to apply

Load the BAM evaluation results from the Zenodo deposit (DOI 10.5281/zenodo.13903946) and the SIRIUS 5.6.3 baseline results computed on the same 60,368 anchor–suspect pair data set, keeping one Boolean correctness label per query per method (for SIRIUS, correctness was judged after molecular formula annotation, with structure annotation only if the correct formula was generated). Restrict to query spectra shared by both runs, then build a 2x2 contingency table crossing BAM correctness against SIRIUS correctness using pandas/scikit-learn in the Python 3.9 environment. Compute the counts of queries correct in both methods, correct only in BAM, and correct only in SIRIUS. Plot the uniqueness breakdown with Matplotlib/Seaborn (bar or Venn-style), save the contingency table as CSV and the figure as PNG, and verify that the majority of correct queries are unique to each method, confirming complementary annotation results.

## Related tools

- **BAM** (Method under evaluation; its per-query correctness labels come from the Zenodo deposit) — https://github.com/HassounLab/BAM
- **SIRIUS 5.6.3** (Baseline method applied to the same 60,368 anchor–suspect pair query spectra to produce comparison correctness labels)
- **pandas / scikit-learn 1.1.2** (Build the correctness contingency table and compute both-correct / method-unique counts)
- **Matplotlib / Seaborn** (Render the bar or Venn-style uniqueness breakdown of complementary correct-annotation counts)

## Examples

```
import pandas as pd; ct = pd.crosstab(bam_results["query"].map(bam_correct), sirius_results["query"].map(sirius_correct)); ct.to_csv("bam_sirius_contingency.csv")
```

## Evaluation signals

- Contingency table dimensions are 2x2 and cell counts sum to the number of query spectra shared by both runs.
- Counts of correct-in-both, correct-only-in-BAM, and correct-only-in-SIRIUS are mutually exclusive and exhaust the correct-annotation set.
- The majority of correct queries fall into method-unique cells (BAM-only or SIRIUS-only), matching the paper's complementarity finding.
- Both output artifacts exist and are valid: the contingency CSV parses with pandas and the PNG is a non-empty Matplotlib/Seaborn figure.
- SIRIUS correctness labels reflect the paper's two-stage criterion: formula annotation first, structure annotation only if the correct formula was generated.

## Limitations

- Comparison is restricted to query spectra shared by both runs; queries evaluated by only one method are excluded from the contingency table.
- Correctness labels are Boolean per query, so the analysis captures overlap of correct annotations but not rank or score-level agreement between methods.

## Evidence

- [methods] SIRIUS applied to same query set: [evidence span withheld — anchor status: not_found]
- [methods] Two-stage correctness criterion: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [results] Query set provenance: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [methods] Results deposit: "All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946)."
- [abstract] BAM headline accuracy context: "BAM was able to assign correct molecular structures to 24.2% of examined anchor-suspect cases"
