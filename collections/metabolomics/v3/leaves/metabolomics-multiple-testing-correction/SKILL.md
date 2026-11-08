---
name: metabolomics-multiple-testing-correction
description: 'Use when a metabolomics pipeline needs to control false discoveries:
  selecting differentially abundant metabolites from two-tailed t-tests, or converting
  raw ORA pathway p-values into q-values before declaring significant enriched pathways
  (SEPs).'
when_to_use_negative:
- When reporting exploratory results without correction is explicitly intended — the
  article flags that 'a p-value threshold of P <= 0.1 was used without multiple testing
  correction to select SEPs' as a limitation, not a recommended default
- When the goal is a single pre-specified pathway hypothesis test rather than simultaneous
  testing of a full KEGG/Reactome/BioCyc pathway set
- When the input is already a table of adjusted q-values — re-applying BH correction
  would double-adjust and over-conservatively shrink the SEP list
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3173
tools:
- name: scipy stats fisher_exact
  role: Computes the raw right-tailed (hypergeometric) per-pathway p-values that the
    BH FDR procedure then corrects into q-values
  repo: github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Hosts the reproducible_simulations.ipynb cells in which the BH FDR correction
    and SEP counting at q ≤ 0.1 are executed and exported to CSV
  repo: github.com/cwieder/metabolomics-ORA
- name: Python 3.8
  role: Runtime environment in which the correction and simulation code was tested
    (MacOS v11.2.3, 16GB RAM)
  repo: github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:77c2d86c5910ab47aab8b43def3d787161d15b7c0a12ddf9f7905f1e967468c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - MetaboAnalyst
  - Python
  - Python 3.8
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolomics-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg False Discovery Rate (BH FDR) correction to metabolomics p-values — both for selecting differentially abundant metabolites prior to ORA (q ≤ 0.05) and for calling significantly enriched pathways from right-tailed Fisher's exact test pathway p-values (q ≤ 0.1).

## When to use

Use when a metabolomics pipeline needs to control false discoveries: selecting differentially abundant metabolites from two-tailed t-tests, or converting raw ORA pathway p-values into q-values before declaring significant enriched pathways (SEPs).

## When NOT to use

- When reporting exploratory results without correction is explicitly intended — the article flags that 'a p-value threshold of P <= 0.1 was used without multiple testing correction to select SEPs' as a limitation, not a recommended default
- When the goal is a single pre-specified pathway hypothesis test rather than simultaneous testing of a full KEGG/Reactome/BioCyc pathway set
- When the input is already a table of adjusted q-values — re-applying BH correction would double-adjust and over-conservatively shrink the SEP list

## Inputs

- Vector of raw per-pathway p-values from right-tailed Fisher's exact test (scipy.stats.fisher_exact, hypergeometric)
- Per-metabolite p-values from two-tailed Student's t-tests on imputed, log2-transformed, auto-scaled abundance matrices
- Pathway p-value table (as tabulated in pvalue_comparison.csv) across background settings
- Significance thresholds: q ≤ 0.05 (differential metabolite selection), p ≤ 0.1 (uncorrected SEP threshold), q ≤ 0.1 (FDR-corrected SEP threshold)

## Outputs

- BH FDR-adjusted q-values for every pathway tested in a dataset
- Count of significantly enriched pathways (SEPs) at q ≤ 0.1 after FDR correction (sep_count_comparison.csv)
- Selected list of differentially abundant metabolites at BH FDR q ≤ 0.05, used as the metabolites of interest for ORA
- Side-by-side comparison of SEP counts at uncorrected p ≤ 0.1 vs FDR-corrected q ≤ 0.1

## How to apply

First compute raw per-pathway p-values using scipy.stats.fisher_exact (right-tailed, hypergeometric), calculating a p-value only when k ≥ 1 metabolites of interest map to the pathway. Then apply the Benjamini-Hochberg FDR procedure across all pathway p-values within a dataset to obtain q-values, and count SEPs at q ≤ 0.1; also tabulate uncorrected p ≤ 0.1 counts so the effect of correction can be quantified (as in the paper's sep_count_comparison.csv). For the metabolite-of-interest list, run two-tailed Student's t-tests on auto-scaled log2-transformed abundances and retain metabolites at BH FDR q ≤ 0.05 before running ORA. The rationale is that correction substantially reduces false positives: the paper found far fewer SEPs after FDR correction than at uncorrected p ≤ 0.1, with two datasets dropping to zero SEPs. Implement this inside the reproducible_simulations.ipynb Jupyter notebook (Python 3.8), saving the corrected and uncorrected results side by side for comparison.

## Related tools

- **scipy stats fisher_exact** (Computes the raw right-tailed (hypergeometric) per-pathway p-values that the BH FDR procedure then corrects into q-values) — github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Hosts the reproducible_simulations.ipynb cells in which the BH FDR correction and SEP counting at q ≤ 0.1 are executed and exported to CSV) — github.com/cwieder/metabolomics-ORA
- **Python 3.8** (Runtime environment in which the correction and simulation code was tested (MacOS v11.2.3, 16GB RAM)) — github.com/cwieder/metabolomics-ORA

## Examples

```
from scipy.stats import fisher_exact; p = fisher_exact([[k, M-k], [n-k, N-M-n+k]], alternative='greater')[1]  # raw pathway p-value, then BH-adjust across pathways
```

## Evaluation signals

- SEP count at q ≤ 0.1 after BH FDR correction must be less than or equal to the SEP count at uncorrected p ≤ 0.1 for every dataset and background setting
- Under the assay-specific background, two of the five datasets (Quirós et al. 2017 and Fuhrer et al. 2017) should show zero SEPs after q ≤ 0.1 correction
- Q-values must be non-decreasing when raw p-values are sorted ascending, and every q ≥ its corresponding raw p-value
- The metabolite-of-interest list must be a strict subset of the background set after selection at q ≤ 0.05
- Results reproduce the pvalue_comparison.csv and sep_count_comparison.csv table structure with one row per pathway/dataset and columns for both significance criteria

## Limitations

- In the majority of the paper's examples, a p-value threshold of P ≤ 0.1 was used without multiple testing correction to select SEPs, so uncorrected comparisons overstate enrichment
- The study lacked a ground-truth dataset where the identities of enriched pathways are known, so correction levels cannot be validated against true positives
- Defining the ideal assay-specific background set remains an open problem, and the choice of background interacts with how correction affects the SEP count

## Evidence

- [discussion] P-values were adjusted using the Benjamini-Hochberg False discovery rate (BH FDR) procedure: "P-values were adjusted using the Benjamini-Hochberg False discovery rate (BH FDR) procedure"
- [discussion] Significantly differentially abundant (DA) metabolites were then selected based on a q-value threshold of q <= 0.05: [evidence span withheld — anchor status: not_found]
- [intro] P-values for each pathway are calculated using a right-tailed Fisher's exact test based on the hypergeometric distribution.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [discussion] Another limitation is that in the majority of examples, a p-value threshold of P <= 0.1 was used without multiple testing correction to select SEPs.: [evidence span withheld — anchor status: not_found]
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [readme] The Python code to generate the results is contained within the Jupyter notebook src/reproducible_simulations.ipynb.: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
