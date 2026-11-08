---
name: kegg-pathway-set-property-characterization
description: Use when deciding between the KEGG organism-specific human compound-only
  pathway set (261 pathways, mean size 30) and the KEGG reference compound-only set
  (377 pathways, mean size 45) for ORA on metabolomics data, and you need quantitative
  evidence of how much more/larger pathways in one set drive.
when_to_use_negative:
- When the goal is to compare pathway databases other than KEGG (e.g., Reactome or
  BioCyc) — the article states an extensive comparison of pathway databases is beyond
  the scope of the paper.
- When the task is to simulate metabolite misidentification or evaluate its effect
  on ORA results — this skill only characterizes static pathway set properties, not
  misidentification-driven changes.
- When a ground-truth set of enriched pathways is available and you can evaluate ORA
  accuracy directly — the authors note no such ground-truth dataset exists, so this
  skill cannot be validated against known enriched pathways.
edam_operation: http://edamontology.org/operation_0270
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: Python 3.8
  role: Implementation language for the reproducible_simulations.ipynb notebook that
    computes pathway set properties and overlap coefficients.
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to compute ORA p-values per pathway,
    needed to produce the SEP counts that complement the pathway set property statistics.
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Execution environment for the reproducible_simulations.ipynb script that generates
    pathway_set_properties.csv and sep_counts_table.csv.
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:10228fff85711a1431a6cf60df356f10bf47bd695f376de7c470f168425d51a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_0270
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# kegg-pathway-set-property-characterization

## Summary

Characterize and compare the structural properties of alternative KEGG pathway sets (organism-specific vs. reference) — pathway counts, mean sizes, and overlap coefficients — before choosing one for over-representation analysis (ORA) in metabolomics.

## When to use

Use when deciding between the KEGG organism-specific human compound-only pathway set (261 pathways, mean size 30) and the KEGG reference compound-only set (377 pathways, mean size 45) for ORA on metabolomics data, and you need quantitative evidence of how much more/larger pathways in one set drive SEP counts.

## When NOT to use

- When the goal is to compare pathway databases other than KEGG (e.g., Reactome or BioCyc) — the article states an extensive comparison of pathway databases is beyond the scope of the paper.
- When the task is to simulate metabolite misidentification or evaluate its effect on ORA results — this skill only characterizes static pathway set properties, not misidentification-driven changes.
- When a ground-truth set of enriched pathways is available and you can evaluate ORA accuracy directly — the authors note no such ground-truth dataset exists, so this skill cannot be validated against known enriched pathways.

## Inputs

- processed compound lists (metabolites of interest and background sets) for the five datasets (MTBLS135, Yachida et al. 2019 stool, MTBLS136, Quiros et al. 2017, Fuhrer et al. 2017)
- KEGG human compound-only pathway set (261 pathways, mean size 30)
- KEGG reference compound-only pathway set (377 pathways, mean size 45)

## Outputs

- pathway_set_properties.csv (pathway counts, mean sizes, overlap coefficients)
- sep_counts_table.csv (significantly enriched pathways per dataset per set, with reference-only flags)

## How to apply

Launch the reproducible_simulations.ipynb Jupyter notebook (Python 3.8) and load the processed compound lists — metabolites of interest and background sets — for the five study datasets (MTBLS135, Yachida et al. 2019 stool, MTBLS136, Quiros et al. 2017, Fuhrer et al. 2017). Compute basic property statistics of the KEGG human compound-only pathway set (261 pathways, mean size 30) and the KEGG reference compound-only set (377 pathways, mean size 45), including overlap coefficients (OC) for shared-identifier pathway pairs, and write them to pathway_set_properties.csv. Then, for each dataset, run right-tailed Fisher's exact test (scipy.stats.fisher_exact) ORA restricted to pathways with at least 3 compounds, calculating a p-value when k (differentially abundant metabolites in the pathway) >= 1, and count significantly enriched pathways (SEPs) at p <= 0.1 without multiple testing correction to produce sep_counts_table.csv. Validate that pathway_set_properties.csv reproduces the median OC of 0.92 (IQR 0.83–0.97) for shared-ID pathway pairs and that sep_counts_table.csv shows the excess of reference-only SEPs reported in the article.

## Related tools

- **Python 3.8** (Implementation language for the reproducible_simulations.ipynb notebook that computes pathway set properties and overlap coefficients.) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to compute ORA p-values per pathway, needed to produce the SEP counts that complement the pathway set property statistics.) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Execution environment for the reproducible_simulations.ipynb script that generates pathway_set_properties.csv and sep_counts_table.csv.) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- pathway_set_properties.csv must show a median overlap coefficient (OC) of 0.92 with IQR 0.83–0.97 for shared-identifier pathway pairs between the KEGG human and KEGG reference sets.
- pathway_set_properties.csv must reflect the documented pathway counts and mean sizes: KEGG human = 261 pathways / mean size 30, KEGG reference = 377 pathways / mean size 45.
- sep_counts_table.csv must show many more significantly enriched pathways (SEPs) using the KEGG reference set alone than with the organism-specific set alone across the five datasets.
- sep_counts_table.csv must flag pathways enriched only with the reference set, matching the excess of reference-only SEPs reported in the article.
- All computations must be reproducible via the reproducible_simulations.ipynb notebook run under Python 3.8.

## Limitations

- No ground-truth dataset is available where the identities of enriched pathways are known, so the accuracy of the SEP comparison cannot be validated.
- In the majority of examples a p <= 0.1 threshold was used without multiple testing correction to select SEPs.
- The study is limited to five MS-based untargeted metabolomics datasets.
- Comparison of pathway databases is not performed; readers are referred to other studies.

## Evidence

- [methods] Compute basic property statistics of the KEGG human compound-only pathway set (261 pathways, mean size 30) and the KEGG reference compound-only set (377 pathways, mean size 45), including overlap coefficients: [evidence span withheld — anchor status: not_found]
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: [evidence span withheld — anchor status: not_found]
- [readme] All code has been tested using Python 3.8 on MacOS (v11.2.3) with standard hardware (16GB RAM).: "All code has been tested using Python 3.8 on MacOS (v11.2.3) with standard hardware (16GB RAM)."
- [discussion] This study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known.: "This study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known."
