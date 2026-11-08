---
name: kegg-pathway-set-filtering
description: Use when assembling pathway sets from KEGG (or Reactome/BioCyc) as ORA
  input for metabolomics data, to exclude pathways with fewer than 3 compounds from
  the over-representation analysis.
when_to_use_negative:
- The goal is pathway-database comparison or visualisation rather than ORA, where
  pathway-size summaries may themselves be of interest rather than a filter criterion
- The pathway collection is a custom metabolite set designed by the user with intentionally
  small sets (fewer than 3 metabolites), which the >=3-compound rule would discard
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3828
tools:
- name: MetaboAnalyst 4.0
  role: Converts compound names in each dataset to KEGG identifiers so compounds can
    be mapped to KEGG pathway sets prior to filtering
- name: reproducible_simulations.ipynb (Jupyter Notebook)
  role: Runs the ORA pipeline in which the >=3-compound pathway filter is applied
    before right-tailed scipy.stats.fisher_exact testing
  repo: github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:a722d7bcffdd3fd036b67efa8eb6e8b00934222c4670eb13938c7cf61136fe78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - pip3
  - reproducible_simulations.ipynb (Jupyter Notebook)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3828
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kegg-pathway-set-filtering

## Summary

Pre-filters KEGG pathway sets (e.g., KEGG release 96 human pathway sets) to retain only pathways containing at least 3 mapped compounds before running ORA, so that Fisher's exact tests are not computed for degenerate, nearly-empty pathway sets.

## When to use

Use when assembling pathway sets from KEGG (or Reactome/BioCyc) as ORA input for metabolomics data, to exclude pathways with fewer than 3 compounds from the over-representation analysis.

## When NOT to use

- The goal is pathway-database comparison or visualisation rather than ORA, where pathway-size summaries may themselves be of interest rather than a filter criterion
- The pathway collection is a custom metabolite set designed by the user with intentionally small sets (fewer than 3 metabolites), which the >=3-compound rule would discard

## Inputs

- KEGG pathway sets (e.g., KEGG release 96 human pathway sets) as compound lists per pathway
- Dataset compounds mapped to KEGG identifiers (via MetaboAnalyst 4.0 ID conversion)
- Assay-specific or nonspecific KEGG background compound list (e.g., the 3373-compound KEGG human background)

## Outputs

- Filtered pathway set containing only pathways with >= 3 compounds, used as ORA input
- Per-pathway p-value table (e.g., pvalue_comparison.csv) computed only for qualifying pathways

## How to apply

Convert compound names in each dataset to KEGG identifiers (e.g., with the MetaboAnalyst 4.0 ID conversion tool) and map them onto the KEGG release 96 human pathway sets. Count how many mapped compounds each pathway contains and discard pathways containing fewer than 3 compounds; the article states that 'Only pathways containing at least 3 compounds were used as input for ORA.' Apply the filter before building the background/reference compound list and before running the right-tailed Fisher's exact test (hypergeometric) ORA in the reproducible_simulations.ipynb Jupyter notebook. During the ORA itself, additionally compute a pathway p-value only when k >= 1 metabolites of interest map to the pathway, so the two filters together avoid spurious p-values for very small or empty pathway sets. The same ≥3-compound rule is part of the paper's minimal reporting/recommendation guidance for reproducible ORA in metabolomics.

## Related tools

- **MetaboAnalyst 4.0** (Converts compound names in each dataset to KEGG identifiers so compounds can be mapped to KEGG pathway sets prior to filtering)
- **reproducible_simulations.ipynb (Jupyter Notebook)** (Runs the ORA pipeline in which the >=3-compound pathway filter is applied before right-tailed scipy.stats.fisher_exact testing) — github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Every pathway entering ORA has at least 3 compounds mapped from the background set; no pathway with < 3 compounds yields a p-value
- p-values are only computed for pathways with k >= 1 differentially abundant metabolites mapped
- The filtered pathway count is consistent with KEGG release 96 human pathway sets intersected with the mapped compound space
- Running the notebook end-to-end (local Python 3.8 or Google Colab) reproduces the tabulated p-value and SEP-count outputs (pvalue_comparison.csv, sep_count_comparison.csv)

## Limitations

- No ground-truth dataset is available where the identities of enriched pathways are known, so the impact of the filter choice cannot be validated against truth
- Filtering interacts with the choice of background set: using a non-assay-specific background (e.g., the KEGG human 3373-compound set) inflates false-positive pathways, so the >=3-compound filter alone does not prevent ORA artefacts
- The ≥3-compound cutoff is a convention from this study; no benchmark justification or sensitivity analysis of the threshold itself is provided

## Evidence

- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: [evidence span withheld — anchor status: not_found]
- [results] Assemble the KEGG release 96 human pathway sets and retain only pathways containing at least 3 compounds as ORA input.: [evidence span withheld — anchor status: not_found]
- [intro] P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
