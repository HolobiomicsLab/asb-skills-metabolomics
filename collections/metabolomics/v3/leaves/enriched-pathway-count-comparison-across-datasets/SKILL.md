---
name: enriched-pathway-count-comparison-across-datasets
description: Decision support for choosing between an organism-specific and a reference
  (non-organism-specific) pathway database set when running ORA on metabolomics data;
when_to_use_negative:
- The user has already committed to a single pathway set and only needs a one-shot
  enrichment result — this skill is for the A/B comparison between an organism-specific
  and reference set, not for the ORA run itself.
- The pathway database provides no organism-specific variant (e.g. Reactome or BioCyc
  as used in this paper), so there is no pair to compare.
- A ground-truth list of truly enriched pathways is required — the paper explicitly
  notes no ground-truth dataset exists for identifying which enriched pathways are
  correct, so the count comparison cannot arbitrate which set is biologically right.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: scipy stats fisher_exact
  role: Implements the right-tailed Fisher's exact (hypergeometric) test that generates
    the per-pathway p-values whose significant hits are counted for each pathway set
- name: Jupyter Notebook (reproducible_simulations.ipynb)
  role: Contains the code cells that load the processed compound lists, compute pathway-set
    properties, run the paired ORA, and assemble the SEP count table
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:c60681c8c20e52b5484aee67219ac85207fccefbd1265517c3f9aa053f99a846
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Jupyter Notebook (reproducible_simulations.ipynb)
  - Python
  - scipy stats fisher_exact
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# enriched-pathway-count-comparison-across-datasets

## Summary

Compares counts of significantly enriched pathways (SEPs) obtained from over-representation analysis (right-tailed Fisher's exact test / hypergeometric) when run against two competing pathway sets — here the KEGG human compound-only set (261 pathways, mean size 30) versus the KEGG reference compound-only set (377 pathways, mean size 45) — across five metabolomics datasets, quantifying the inflation of pathway hits caused by choosing the larger, more generic reference set.

## When to use

Decision support for choosing between an organism-specific and a reference (non-organism-specific) pathway database set when running ORA on metabolomics data; triggered when a pathway database offers both variants and the user must judge how many extra significant pathways the larger set will produce.

## When NOT to use

- The user has already committed to a single pathway set and only needs a one-shot enrichment result — this skill is for the A/B comparison between an organism-specific and reference set, not for the ORA run itself.
- The pathway database provides no organism-specific variant (e.g. Reactome or BioCyc as used in this paper), so there is no pair to compare.
- A ground-truth list of truly enriched pathways is required — the paper explicitly notes no ground-truth dataset exists for identifying which enriched pathways are correct, so the count comparison cannot arbitrate which set is biologically right.

## Inputs

- Metabolites-of-interest list (differentially abundant compounds) per dataset
- Background/reference compound list per dataset
- KEGG organism-specific compound-only pathway set (261 pathways, mean size 30 for human)
- KEGG reference compound-only pathway set (377 pathways, mean size 45)
- Pathway set property statistics (pathway count, mean size, overlap coefficients for shared-identifier pairs)

## Outputs

- Per-dataset counts of significantly enriched pathways (SEPs) at p ≤ 0.1 for each pathway set (sep_counts_table.csv analogue)
- List of pathways enriched only with the reference set
- Pathway set property statistics table (pathway_set_properties.csv analogue) with median overlap coefficient for shared-ID pairs

## How to apply

First characterise both pathway sets in the reproducible_simulations.ipynb notebook: compute the number of pathways, mean pathway size, and overlap coefficients (OC) for shared-identifier pathway pairs — the article's data show a median OC of 0.92 (IQR 0.83–0.97) between KEGG human and KEGG reference pathway pairs. Then, for each of the five study datasets (MTBLS135, Yachida et al. 2019 stool, MTBLS136, Quiros et al. 2017, Fuhrer et al. 2017), run the identical ORA implementation — scipy stats fisher_exact, right-tailed/hypergeometric, restricted to pathways with ≥3 compounds, with a p-value only when k ≥ 1 — against each pathway set using the same metabolites-of-interest and background list. Count SEPs per dataset per set at p ≤ 0.1 without multiple testing correction, and assemble a comparison table flagging pathways significant only with the reference set; the expectation, confirmed in the article, is far more SEPs with the reference set despite large overlap between analogous pathways.

## Related tools

- **scipy stats fisher_exact** (Implements the right-tailed Fisher's exact (hypergeometric) test that generates the per-pathway p-values whose significant hits are counted for each pathway set)
- **Jupyter Notebook (reproducible_simulations.ipynb)** (Contains the code cells that load the processed compound lists, compute pathway-set properties, run the paired ORA, and assemble the SEP count table) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
jupyter-notebook reproducible_simulations.ipynb  # cd metabolomics-ORA/src first
```

## Evaluation signals

- The computed pathway set properties should reproduce the article's median overlap coefficient of 0.92 (IQR 0.83–0.97) for shared-identifier pathway pairs between KEGG human and KEGG reference sets
- Pathway counts and mean sizes must match the stated values: 261 pathways / mean size 30 for the KEGG human set and 377 pathways / mean size 45 for the KEGG reference set
- For every dataset, the number of SEPs at p ≤ 0.1 must be equal or larger with the reference set than with the organism-specific set — any reversal indicates a pipeline error
- The same metabolites-of-interest and background list must be used for both pathway-set runs so any SEP-count difference is attributable only to the pathway set
- SEPs must be selected at p ≤ 0.1 without multiple testing correction, matching the article's stated threshold for this comparison

## Limitations

- No ground-truth dataset exists to judge which of the two SEP lists is biologically correct — the comparison shows counts differ but cannot say which set is right
- The p ≤ 0.1 threshold is used without multiple testing correction, so the SEP counts are not conservative significance calls
- Results are limited to five MS-based untargeted metabolomics datasets; generalisation to other data types is untested
- The reference set contains both more and larger pathways, so count inflation may partially reflect set size rather than pathway content

## Evidence

- [results] Organism-specific vs reference pathway-set comparison finding: [evidence span withheld — anchor status: not_found]
- [results] Pathway set sizes: [evidence span withheld — anchor status: not_found]
- [methods] ORA implementation details: [evidence span withheld — anchor status: not_found]
- [results] SEP threshold: [evidence span withheld — anchor status: not_found]
- [results] Overlap statistics validation: [evidence span withheld — anchor status: not_found]
- [discussion] Five datasets used: [evidence span withheld — anchor status: not_found]
- [readme] Launch instructions: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [discussion] ORA implementation tool: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
