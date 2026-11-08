---
name: annotation-correctness-overlap-analysis
description: Use when two annotation/ prediction methods have been run on the same
  query spectra and you need to quantify overlap vs. complementarity of their correct-annotation
  sets (e.g., BAM vs. SIRIUS on 60,368 anchor–suspect pairs).
when_to_use_negative:
- You only have correctness labels from a single method and no second-method baseline
  to compare against
- The two methods were run on different query sets with little or no shared query-spectrum
  identifier space, so the intersection is not meaningful
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 5.6.3
  role: Baseline molecular formula and structure annotation method applied to the
    same query spectra; its correctness labels form the comparison set
- name: Pandas
  role: Building the contingency table and computing correct-only / both-correct counts
    over shared queries
- name: scikit-learn 1.1.2
  role: Used in the Python 3.9 analysis environment for computing correctness cross-tabulation
    statistics
- name: Matplotlib
  role: Rendering the uniqueness breakdown figure (bar or Venn-style) saved as PNG
- name: Seaborn
  role: Styling the complementary-annotation breakdown plot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:fb000fcad366e073c8ddf95154913009a7e8bf55dd8431dfce6ea08d8f292e0d
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# annotation-correctness-overlap-analysis

## Summary

Compares per-query Boolean correctness labels from two annotation methods (e.g., biotransformation-based BAM and SIRIUS 5.6.3 structure annotation) on the same evaluation set, building a contingency table and uniqueness breakdown to determine whether the methods provide complementary annotations.

## When to use

Use when two annotation/ prediction methods have been run on the same query spectra and you need to quantify overlap vs. complementarity of their correct-annotation sets (e.g., BAM vs. SIRIUS on 60,368 anchor–suspect pairs).

## When NOT to use

- You only have correctness labels from a single method and no second-method baseline to compare against
- The two methods were run on different query sets with little or no shared query-spectrum identifier space, so the intersection is not meaningful

## Inputs

- BAM evaluation results file with per-query Boolean correctness labels (from Zenodo deposit 10.5281/zenodo.13903946)
- SIRIUS 5.6.3 baseline results with per-query Boolean correctness labels on the same 60,368 anchor–suspect pair data set
- Shared query spectrum identifiers linking both result sets

## Outputs

- Contingency table CSV crossing BAM correctness vs. SIRIUS correctness
- Uniqueness breakdown figure (PNG) showing correct-only-in-BAM, correct-only-in-SIRIUS, and correct-in-both counts
- Overlap counts: correct in both, correct only in BAM, correct only in SIRIUS

## How to apply

Load per-query correctness labels for both methods — BAM evaluation results from the Zenodo deposit (DOI 10.5281/zenodo.13903946) and SIRIUS 5.6.3 baseline results on the same 60,368 anchor–suspect pair data set — keeping a Boolean correct/incorrect label per query spectrum per method. Restrict to query spectra shared by both runs and cross BAM correctness against SIRIUS correctness in a contingency table using pandas/scikit-learn in a Python 3.9 environment. Compute three disjoint counts: queries correct in both methods, correct only in BAM, and correct only in SIRIUS. Visualize the uniqueness breakdown (bar or Venn-style) with Matplotlib/Seaborn, and export the contingency table as CSV and the figure as PNG. The evaluation succeeds if the majority of correct queries are unique to each method, confirming that biotransformation-based annotation and SIRIUS formula/structure annotation are complementary rather than redundant.

## Related tools

- **SIRIUS 5.6.3** (Baseline molecular formula and structure annotation method applied to the same query spectra; its correctness labels form the comparison set)
- **Pandas** (Building the contingency table and computing correct-only / both-correct counts over shared queries)
- **scikit-learn 1.1.2** (Used in the Python 3.9 analysis environment for computing correctness cross-tabulation statistics)
- **Matplotlib** (Rendering the uniqueness breakdown figure (bar or Venn-style) saved as PNG)
- **Seaborn** (Styling the complementary-annotation breakdown plot)

## Examples

```
import pandas as pd; ct = pd.crosstab(bam_eval['query_id'].map(labels), sirius_eval['correct'])
```

## Evaluation signals

- The three computed counts (correct in both, correct only in BAM, correct only in SIRIUS) sum exactly to the number of shared query spectra
- The contingency table is a 2x2 Boolean cross-tabulation with no missing/ambiguous labels after the shared-query intersection
- The exported CSV row/column totals match the per-method totals of correct annotations computed independently
- The PNG breakdown figure's bar heights match the CSV contingency counts
- The reported overlap reproduces the paper's conclusion that the majority of correct queries are unique to each method

## Limitations

- Correctness labels are only available on queries where both methods produced outputs; the shared-query intersection may exclude queries handled by one method only
- SIRIUS baseline correctness was gated: structure annotation was only attempted if molecular formula annotation first produced the correct formula
- Results are specific to the 60,368 anchor–suspect pair evaluation set derived from the global molecular network; generalization to other data sets is not established by this analysis

## Evidence

- [methods] we applied SIRIUS (version 5.6.3) to the query spectra represented in the same data set of 60,368 anchor–suspect pairs.: [evidence span withheld — anchor status: not_found]
- [results] when comparing correct queries from BAM and SIRIUS, the majority of correct queries are unique to each method: [evidence span withheld — anchor status: not_found]
- [methods] we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation.: "we first performed molecular formula annotation, and then only if the correct formula was generated, we performed molecular structure annotation."
- [methods] All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946).: "All code, data, and results relevant to this analysis can be found on Zenodo (DOI:10.5281/zenodo.13903946)."
- [methods] Python 3.9, PubChemPy (version 1.0.4), BioServices (version 1.11.2), scikit-learn (version 1.1.2): [evidence span withheld — anchor status: not_found]
