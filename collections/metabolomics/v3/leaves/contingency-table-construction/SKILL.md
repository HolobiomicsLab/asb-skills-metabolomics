---
name: contingency-table-construction
description: Use when two annotation or prediction methods have been run on the same
  query spectra set and you need to quantify overlap vs. uniqueness of correct results
  to support a complementarity claim.
when_to_use_negative:
- The two methods were evaluated on different query sets with no shared query spectra
  to intersect
- Per-query correctness labels are unavailable (e.g., only aggregate accuracy percentages
  were reported)
- The goal is pairwise correlation of continuous scores rather than a binary correct/incorrect
  cross-tabulation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Python 3.9
  role: Runtime environment for loading, cross-tabulating, and plotting correctness
    labels
- name: Pandas
  role: Building the contingency table by crossing BAM and SIRIUS correctness columns
    over shared query spectra
- name: NumPy
  role: Array operations on Boolean correctness vectors
- name: scikit-learn 1.1.2
  role: Confusion-matrix/contingency computation over the shared evaluation set
- name: Matplotlib
  role: Plotting the uniqueness breakdown as a bar or Venn-style figure
- name: Seaborn
  role: Styling the complementarity breakdown figure
- name: SIRIUS 5.6.3
  role: Source of the baseline method's correctness labels for the comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:a2a45b191e92e6fc9d8b31566de27d725b8f02191f568c45b0a801653054d5d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matplotlib
  - NumPy
  - PROXIMAL2
  - Pandas
  - Python 3.9
  - SIRIUS 5.6.3
  - Seaborn
  - scikit-learn 1.1.2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# contingency-table-construction

## Summary

Builds a 2x2 contingency table crossing Boolean correctness labels from two annotation methods (e.g., BAM vs SIRIUS 5.6.3) over a shared query set, then decomposes agreement into correct-in-both, correct-only-in-method-A, and correct-only-in-method-B counts to test complementarity.

## When to use

Use when two annotation or prediction methods have been run on the same query spectra set and you need to quantify overlap vs. uniqueness of correct results to support a complementarity claim.

## When NOT to use

- The two methods were evaluated on different query sets with no shared query spectra to intersect
- Per-query correctness labels are unavailable (e.g., only aggregate accuracy percentages were reported)
- The goal is pairwise correlation of continuous scores rather than a binary correct/incorrect cross-tabulation

## Inputs

- Per-query Boolean correctness labels from BAM evaluation results (Zenodo DOI 10.5281/zenodo.13903946)
- Per-query Boolean correctness labels from SIRIUS 5.6.3 baseline results on the same 60,368 anchor–suspect pair data set

## Outputs

- Contingency table of BAM correctness vs SIRIUS correctness (CSV file)
- Complementarity breakdown figure showing correct-in-both, BAM-only, and SIRIUS-only counts (PNG file)

## How to apply

Load per-query Boolean correctness labels from both evaluation runs — e.g., BAM results from the Zenodo deposit (DOI 10.5281/zenodo.13903946) and SIRIUS 5.6.3 baseline results — restricted to the same 60,368 anchor–suspect pair data set. Restrict to query spectra shared by both runs and cross BAM correctness against SIRIUS correctness using pandas DataFrames with scikit-learn/pandas in a Python 3.9 environment. Compute the three complementarity counts: queries correct in both methods, correct only in BAM, and correct only in SIRIUS. Save the contingency table as CSV and visualize the uniqueness breakdown (bar or Venn-style) with Matplotlib/Seaborn, then check whether the majority of correct queries are unique to each method, which supports the paper's claim that biotransformation-based annotation and SIRIUS are complementary.

## Related tools

- **Python 3.9** (Runtime environment for loading, cross-tabulating, and plotting correctness labels)
- **Pandas** (Building the contingency table by crossing BAM and SIRIUS correctness columns over shared query spectra)
- **NumPy** (Array operations on Boolean correctness vectors)
- **scikit-learn 1.1.2** (Confusion-matrix/contingency computation over the shared evaluation set)
- **Matplotlib** (Plotting the uniqueness breakdown as a bar or Venn-style figure)
- **Seaborn** (Styling the complementarity breakdown figure)
- **SIRIUS 5.6.3** (Source of the baseline method's correctness labels for the comparison)

## Examples

```
import pandas as pd; ct = pd.crosstab(df['bam_correct'], df['sirius_correct']); ct.to_csv('bam_sirius_contingency.csv')
```

## Evaluation signals

- The contingency table's total row count equals the number of query spectra shared by both runs (should match the 60,368 anchor–suspect pair data set after intersection)
- correct-in-both + correct-only-in-BAM + correct-only-in-SIRIUS + correct-in-neither equals the shared query count
- The sum correct-in-both + correct-only-in-BAM equals BAM's standalone correct count, and likewise for SIRIUS (marginal consistency check)
- The finding reproduces the paper's result that the majority of correct queries are unique to each method
- CSV contains one row per (BAM_correct, SIRIUS_correct) combination and the PNG renders the three complementarity counts

## Limitations

- The comparison is only valid on query spectra evaluated by both methods; differences in coverage or filtering between runs bias the overlap counts
- BAM's correctness is defined on the 24.2% of anchor–suspect cases it annotated correctly, so correctness labels for unannotated queries must be handled consistently
- The complementarity conclusion applies to this specific anchor–suspect evaluation set and SIRIUS 5.6.3 configuration, not to SIRIUS generally

## Evidence

- [other] Build a contingency table crossing BAM correctness against SIRIUS correctness over the query spectra shared by both runs.: [evidence span withheld — anchor status: not_found]
- [other] Complementary annotation finding: [evidence span withheld — anchor status: not_found]
- [methods] SIRIUS baseline on same data set: [evidence span withheld — anchor status: not_found]
- [methods] Python environment for analysis: [evidence span withheld — anchor status: not_found]
- [methods] Zenodo deposit location: "All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946)."
