---
name: ora-background-set-parameterization
description: Decision support for choosing and reporting the background/reference
  compound set before running ORA on a list of differentially abundant metabolites.
when_to_use_negative:
- The analysis goal is pathway database comparison rather than background-set selection;
  the paper states an extensive comparison of pathway databases is beyond its scope.
- A ground-truth enriched-pathway dataset is available that can be used to evaluate
  ORA outputs directly, since the study explicitly lacked such a benchmark.
- Inputs are raw abundance matrices; post-processing (imputation with minimum/2, log2
  transformation, auto-scaling) and metabolite-identifier harmonisation must precede
  background-set construction.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to calculate pathway p-values against
    the background set
- name: Python 3.8
  role: Language in which ORA was implemented to compute pathway p-values with the
    chosen background
- name: metabolomics-ORA
  role: Reference implementation of the ORA simulations showing the effect of background-set
    parameterization
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:8bf5616c5001191146c60a9964e5fc5ef2331f99d01a572f76f38726247e7a86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Python
  - Python 3.8
  - metabolomics-ORA
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ora-background-set-parameterization

## Summary

Parameterize over-representation analysis (ORA) in metabolomics with an explicit, assay- and pathway-database-matched background compound set, since using a non-assay-specific background inflates false-positive pathways.

## When to use

Decision support for choosing and reporting the background/reference compound set before running ORA on a list of differentially abundant metabolites.

## When NOT to use

- The analysis goal is pathway database comparison rather than background-set selection; the paper states an extensive comparison of pathway databases is beyond its scope.
- A ground-truth enriched-pathway dataset is available that can be used to evaluate ORA outputs directly, since the study explicitly lacked such a benchmark.
- Inputs are raw abundance matrices; post-processing (imputation with minimum/2, log2 transformation, auto-scaling) and metabolite-identifier harmonisation must precede background-set construction.

## Inputs

- Pathway collection (KEGG, Reactome, or BioCyc metabolite sets; pathways with >= 3 compounds)
- List of metabolites of interest (differentially abundant metabolites)
- Background/reference compound set of size N derived from the assay

## Outputs

- Per-pathway ORA p-values (right-tailed Fisher's exact / hypergeometric)
- List of significantly enriched pathways after multiple testing correction
- Reported background-set definition and its size N

## How to apply

Assemble three ORA inputs — a pathway collection (KEGG, Reactome, or BioCyc, restricted to pathways with at least 3 compounds), the metabolites-of-interest list, and a background/reference compound set of size N. Derive the background from the compounds actually measured in the assay (the set of detected metabolites mapped to pathway-database identifiers), not from a generic all-compounds-of-the-database list; the paper shows a non-assay-specific background produced large numbers of false-positive pathways. Pass N explicitly to the right-tailed Fisher's exact test (hypergeometric) so that p-values are computed against the correct denominator, and keep the background identical across the pathway databases and conditions being compared. Report the exact background set and its size N as a minimal reporting requirement, since changes in this parameter alone can yield profoundly different significant-pathway lists.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to calculate pathway p-values against the background set)
- **Python 3.8** (Language in which ORA was implemented to compute pathway p-values with the chosen background)
- **metabolomics-ORA** (Reference implementation of the ORA simulations showing the effect of background-set parameterization) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
python reproducible_simulations.ipynb  # ORA via scipy.stats.fisher_exact(right-tailed) with explicit assay-specific background set of size N
```

## Evaluation signals

- The background set is the set of compounds actually assayed and mapped to pathway-database identifiers, not the full pathway database compound list.
- N (background-set size) is passed explicitly to the hypergeometric/Fisher's exact computation and matches the number of compounds in the reported background.
- Significant-pathway counts change demonstrably when switching between a non-assay-specific and an assay-specific background, consistent with the paper's finding that background choice alters results.
- Only pathways with at least 3 compounds are used as ORA input, and p-values are computed only when k >= 1.

## Limitations

- Defining the ideal assay-specific background set for a particular dataset remains an open problem.
- Changes in the background set, differential metabolite selection, and pathway database can result in profoundly different ORA results, so conclusions are sensitive to these parameters.
- In most examples a p-value threshold of P <= 0.1 was used without multiple testing correction to select significantly enriched pathways, which may overstate the stability of findings.

## Evidence

- [abstract] The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways.: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [abstract] changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
- [intro] To perform ORA, three essential inputs are required: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds.: "To perform ORA, three essential inputs are required: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds."
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA.: "Only pathways containing at least 3 compounds were used as input for ORA."
- [discussion] Defining the ideal assay-specific background set for a particular dataset remains an area for further study.: "Defining the ideal assay-specific background set for a particular dataset remains an area for further study."
