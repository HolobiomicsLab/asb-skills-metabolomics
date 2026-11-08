---
name: metabolomics-over-representation-analysis
description: Use when you have a list of differentially abundant metabolites (e.g.,
  two-tailed t-test with BH FDR q ≤ 0.05) plus an assay-specific background set, and
  need pathway-level functional interpretation with p-value sensitivity checks.
when_to_use_negative:
- The goal is an extensive comparison of pathway databases rather than ORA itself
  — the article states this is 'beyond the scope of this paper'
- No ground-truth set of truly enriched pathways is needed and results must be benchmarked
  against known pathway identities — the article notes no such ground-truth dataset
  is available
- The input is raw metabolomics data needing upstream processing — imputation (minimum
  value / 2), log2 transformation, and auto-scaling must be completed first, and compound
  identifiers converted with the MetaboAnalyst 4.0 ID conversion tool
edam_operation: http://edamontology.org/operation_2436
tools:
- name: scipy stats fisher_exact
  role: Computes right-tailed Fisher's exact (hypergeometric) pathway p-values for
    ORA, only when k ≥ 1 differential metabolites map to the pathway
- name: MetaboAnalyst 4.0
  role: Converts compound names in the datasets to KEGG identifiers prior to ORA
- name: bioservices (Python package) v 1.7.1
  role: Maps KEGG compounds to ChEBI identifiers for the Reactome database arm
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, the notebook implementing all ORA simulations
    (adaptable to user data)
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same notebook without local installation
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:3e82973b9d2eee9fee59b2be0edba29519b36f195cfb5c7fa034dacd51b45c83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - bioservices (Python package) v 1.7.1
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2436
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolomics-over-representation-analysis

## Summary

Performs over-representation analysis (ORA) on metabolomics datasets using a right-tailed Fisher's exact test (hypergeometric) over compound-only pathway sets from KEGG, Reactome, and BioCyc, with explicit control of background-set, pathway-database, and differential-metabolite-threshold parameters. This skill encodes the sensitivity-aware ORA workflow of Wieder et al. 2021 (PLOS Comput Biol).

## When to use

Use when you have a list of differentially abundant metabolites (e.g., two-tailed t-test with BH FDR q ≤ 0.05) plus an assay-specific background set, and need pathway-level functional interpretation with p-value sensitivity checks.

## When NOT to use

- The goal is an extensive comparison of pathway databases rather than ORA itself — the article states this is 'beyond the scope of this paper'
- No ground-truth set of truly enriched pathways is needed and results must be benchmarked against known pathway identities — the article notes no such ground-truth dataset is available
- The input is raw metabolomics data needing upstream processing — imputation (minimum value / 2), log2 transformation, and auto-scaling must be completed first, and compound identifiers converted with the MetaboAnalyst 4.0 ID conversion tool

## Inputs

- Post-harmonised metabolite abundance tables with class labels and differential-abundance calls (KEGG, ChEBI/Reactome, BioCyc identifiers)
- List of differentially abundant metabolites of interest (selected at BH FDR q ≤ 0.05)
- Compound-only pathway sets from KEGG (release 96), Reactome (release 75), and HumanCyc (BioCyc v24.5), filtered to pathways with ≥ 3 compounds
- Assay-specific background set of compounds (and optionally the nonspecific 3373-compound KEGG human background for comparison)

## Outputs

- Per-pathway p-values under each background/database setting (e.g., pvalue_comparison.csv)
- SEP counts per dataset per threshold (p ≤ 0.1 uncorrected, q ≤ 0.1 FDR-corrected; e.g., sep_count_comparison.csv, sep_counts_table.csv)
- Cross-database concordance table with pooled-metabolite overlap coefficients (ora_concordance_table.csv, OC values 0.23–0.62)
- Pathway set property statistics including overlap coefficients for shared-identifier pathway pairs (pathway_set_properties.csv, median OC 0.92, IQR 0.83–0.97)

## How to apply

Prepare three inputs: a collection of compound-only pathway sets (retain only pathways containing at least 3 compounds), a list of metabolites of interest selected at BH FDR q ≤ 0.05, and a background or reference set of compounds. Use an assay-specific background set rather than a nonspecific organism-wide one (e.g., the 3373-compound KEGG human background), since the nonspecific background inflates the number of significantly enriched pathways (SEPs). For each pathway, compute a p-value with scipy.stats.fisher_exact (right-tailed, hypergeometric) only when k ≥ 1 differentially abundant metabolites map to the pathway; call SEPs at p ≤ 0.1 without correction and at q ≤ 0.1 after FDR correction. When comparing pathway databases, harmonise metabolite identifiers to KEGG, Reactome (via ChEBI), and BioCyc, and pool SEP metabolites per database to compute the overlap coefficient |intersection| / min(|pool A|, |pool B|) — expect only low-to-medium consensus (OC 0.23–0.62 across KEGG vs Reactome). The reproducible_simulations.ipynb Jupyter notebook (Python 3.8, also runnable in Google Colab) implements the whole pipeline and can be adapted to user data.

## Related tools

- **scipy stats fisher_exact** (Computes right-tailed Fisher's exact (hypergeometric) pathway p-values for ORA, only when k ≥ 1 differential metabolites map to the pathway)
- **MetaboAnalyst 4.0** (Converts compound names in the datasets to KEGG identifiers prior to ORA)
- **bioservices (Python package) v 1.7.1** (Maps KEGG compounds to ChEBI identifiers for the Reactome database arm)
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, the notebook implementing all ORA simulations (adaptable to user data)) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same notebook without local installation)

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Assay-specific background yields far fewer SEPs than the nonspecific KEGG background at both p ≤ 0.1 and q ≤ 0.1, with two datasets showing zero SEPs after q ≤ 0.1 correction
- Kegg-vs-Reactome overlap coefficients for pooled SEP metabolites fall within 0.23 (Stevens) to 0.62 (Labbé)
- Pathway set properties match reported values: KEGG human = 261 pathways (mean size 30), KEGG reference = 377 pathways (mean size 45), median OC 0.92 (IQR 0.83–0.97) for shared-ID pathway pairs
- Every computed pathway p-value has k ≥ 1 differentially abundant metabolite in the pathway; no p-values are reported for pathways with k = 0
- Only pathways with at least 3 compounds appear as ORA input rows in the output tables

## Limitations

- In most examples a p ≤ 0.1 threshold was used without multiple testing correction to select SEPs
- No ground-truth dataset exists where the identities of enriched pathways are known; simulated ground truths are too simplistic and experimental ones cannot pinpoint exact affected pathways
- Defining the ideal assay-specific background set remains an open problem
- Study limited to five MS-based untargeted metabolomics datasets; Quiros et al. 2017 and Fuhrer et al. 2017 lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications
- Misidentified compounds are unavoidable in practice: no benchmark level of metabolite misidentification exists, and even 4% misidentification causes both gain of false-positive pathways and loss of truly significant pathways

## Evidence

- [discussion] Right-tailed Fisher's exact test ORA restricted to pathways with at least 3 compounds: [evidence span withheld — anchor status: not_found]
- [discussion] ORA implementation via scipy fisher_exact: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [abstract] Nonspecific background causes false positives: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [abstract] Cross-database discordance: "Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results in both the number and function of"
- [readme] Notebook usage instructions: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [discussion] Differential metabolite selection threshold: [evidence span withheld — anchor status: not_found]
