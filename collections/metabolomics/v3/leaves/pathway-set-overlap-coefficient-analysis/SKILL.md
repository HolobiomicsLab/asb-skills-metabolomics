---
name: pathway-set-overlap-coefficient-analysis
description: Use when choosing between candidate pathway databases or pathway sets
  for ORA and you need to know whether differing enrichment counts stem from genuinely
  different biology or merely from set size/overlap differences.
when_to_use_negative:
- When comparing pathway databases that do not share compound identifiers (e.g. KEGG
  IDs vs BioCyc IDs) without prior identifier harmonisation — OC is undefined on non-overlapping
  identifier spaces.
- When the goal is simply to run ORA and report significant pathways; this skill is
  a diagnostic step before or alongside ORA, not a replacement for it.
- When pathway sets are already known to be identical (OC would be trivially 1) and
  the analysis goal lies elsewhere, such as background set selection.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0092
tools:
- name: Python 3.8
  role: Runtime for computing overlap coefficients and pathway set statistics inside
    the notebook
- name: Jupyter Notebook
  role: Hosts reproducible_simulations.ipynb, which contains the OC calculation and
    ORA workflow
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: Used downstream on the same compound lists to link OC-derived set differences
    to ORA p-values (right-tailed, hypergeometric)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:d13c61f9597fad947e07b466b7711b1bd0d443fc15026508231818f1f8feff01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Pathway set overlap coefficient analysis

## Summary

Quantifies how similar two metabolite pathway sets (e.g. organism-specific KEGG human vs KEGG reference) are, pathway by pathway, using the overlap coefficient on shared compound identifiers, to contextualise differences in ORA results across pathway databases.

## When to use

Use when choosing between candidate pathway databases or pathway sets for ORA and you need to know whether differing enrichment counts stem from genuinely different biology or merely from set size/overlap differences.

## When NOT to use

- When comparing pathway databases that do not share compound identifiers (e.g. KEGG IDs vs BioCyc IDs) without prior identifier harmonisation — OC is undefined on non-overlapping identifier spaces.
- When the goal is simply to run ORA and report significant pathways; this skill is a diagnostic step before or alongside ORA, not a replacement for it.
- When pathway sets are already known to be identical (OC would be trivially 1) and the analysis goal lies elsewhere, such as background set selection.

## Inputs

- Processed compound lists — metabolites of interest and background sets — for each study dataset (MTBLS135, Yachida et al. 2019 stool, MTBLS136, Quiros et al. 2017, Fuhrer et al. 2017)
- Two candidate pathway sets restricted to compound-only pathways, each with pathway identifiers and compound members (e.g. KEGG human and KEGG reference)
- Harmonised compound identifiers (e.g. KEGG or ChEBI IDs) shared across both pathway sets

## Outputs

- pathway_set_properties.csv containing per-pair overlap coefficients and set-level summary statistics (number of pathways, mean pathway size)
- Median OC with IQR across shared-identifier pathway pairs (expected: 0.92, IQR 0.83–0.97)
- Set-level counts used to interpret ORA output differences (e.g. 261 KEGG human pathways vs 377 KEGG reference pathways)

## How to apply

Within the reproducible_simulations.ipynb Jupyter notebook, first compute basic property statistics of each candidate pathway set, restricted to compound-only pathways (KEGG human: 261 pathways, mean size 30; KEGG reference: 377 pathways, mean size 45). For every pair of analogous pathways sharing a pathway identifier, calculate the overlap coefficient OC on their compound members and write the results to pathway_set_properties.csv. Then, in a separate step, run ORA on the same compound lists of interest and background sets with the scipy stats fisher_exact function (right-tailed, hypergeometric), restricting input to pathways with at least 3 compounds and calculating a p-value when k >= 1, to link OC values to observed differences in significantly enriched pathway counts. Validation: confirm the computed median OC for shared-ID pathway pairs is 0.92 (IQR 0.83–0.97), matching the value reported by Wieder et al. 2021.

## Related tools

- **Python 3.8** (Runtime for computing overlap coefficients and pathway set statistics inside the notebook)
- **Jupyter Notebook** (Hosts reproducible_simulations.ipynb, which contains the OC calculation and ORA workflow) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Used downstream on the same compound lists to link OC-derived set differences to ORA p-values (right-tailed, hypergeometric))

## Examples

```
jupyter-notebook reproducible_simulations.ipynb  # compute OC in pathway_set_properties.csv, then ORA via scipy.stats.fisher_exact
```

## Evaluation signals

- pathway_set_properties.csv reproduces the median OC of 0.92 (IQR 0.83–0.97) for shared-identifier pathway pairs
- Set-level statistics match the reported values: 261 pathways / mean size 30 for KEGG human and 377 pathways / mean size 45 for the KEGG reference set
- Only pathways with a shared identifier are paired for OC computation; pairs without a common ID are excluded
- OC values fall within [0, 1] and OC = 1 is returned only for identical compound membership
- Downstream ORA p-values are reproducible when the same compound lists of interest and background sets are re-run through scipy stats fisher_exact (right-tailed) on both pathway sets

## Limitations

- OC requires compound identifiers to be harmonised across both pathway sets before any comparison can be made.
- Overlap coefficients describe identifier-set similarity only; they do not capture differences in pathway annotation quality or biological semantics.
- Comparison of pathway databases was not performed extensively in the source study — readers were referred to other dedicated comparison studies — so OC should be treated as a descriptive diagnostic, not a substitute for a full database benchmark.

## Evidence

- [methods] Compute basic property statistics of the KEGG human compound-only pathway set (261 pathways, mean size 30) and the KEGG reference compound-only set (377 pathways, mean size 45), including overlap coefficients (OC) for shared-identifier pathway pairs: [evidence span withheld — anchor status: not_found]
- [results] Many more pathways were significantly enriched using the reference set alone than with the organism-specific set alone, even though the two sets share a large overlap of analogous pathways.: [evidence span withheld — anchor status: not_found]
- [methods] Right-tailed Fisher's exact test (hypergeometric) ORA restricted to pathways with at least 3 compounds: "Only pathways containing at least 3 compounds were used as input for ORA"
- [discussion] For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 ID conversion tool was used: "For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 [47] ID conversion tool was used"
- [abstract] Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results: "Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results"
