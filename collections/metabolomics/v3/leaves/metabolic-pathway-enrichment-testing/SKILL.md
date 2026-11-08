---
name: metabolic-pathway-enrichment-testing
description: Use when a list of differentially abundant metabolites must be interpreted
  functionally via ORA against KEGG, Reactome, or BioCyc compound sets, and background-set/parameter
  choices must be controlled.
when_to_use_negative:
- The goal is an extensive comparison of pathway databases — the authors state this
  is beyond the scope of the paper and refer readers to dedicated comparison studies
- The data are not MS-based untargeted metabolomics compound lists — the study's conclusions
  were validated only on five MS-based untargeted metabolomics datasets
- No assay-specific background set can be defined — a non-assay-specific background
  yields large numbers of false-positive pathways, so conclusions from such a run
  are unreliable
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: scipy stats fisher_exact
  role: Implements the right-tailed (hypergeometric) Fisher's exact test that computes
    each pathway's ORA p-value
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, which contains the Python code for all
    ORA simulations and can be adapted to the user's own data
  repo: github.com/cwieder/metabolomics-ORA
- name: pip3
  role: Installs the notebook dependencies via requirements.txt before launching the
    ORA simulations
  repo: github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative to run the ORA notebook without local installation
  repo: github.com/cwieder/metabolomics-ORA
- name: MetaboAnalyst 4.0
  role: Converts compound names to KEGG identifiers for mapping metabolites into pathway
    compound sets
- name: bioservices (Python package)
  role: Maps KEGG compounds to ChEBI identifiers for Reactome pathway sets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:2533fff0e2d9439783a6e45d564581d7ab86930a023b9e8f61d0220dd1071527
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - MetaboAnalyst 4.0
  - Python
  - bioservices (Python package)
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# metabolic-pathway-enrichment-testing

## Summary

Over-representation analysis (ORA) for metabolomics: tests whether differentially abundant metabolites are over-represented in KEGG, Reactome, or BioCyc pathway compound sets using a right-tailed Fisher's exact (hypergeometric) test, with protocol choices (assay-specific background, ≥3-compound pathways) that control false positives.

## When to use

Use when a list of differentially abundant metabolites must be interpreted functionally via ORA against KEGG, Reactome, or BioCyc compound sets, and background-set/parameter choices must be controlled.

## When NOT to use

- The goal is an extensive comparison of pathway databases — the authors state this is beyond the scope of the paper and refer readers to dedicated comparison studies
- The data are not MS-based untargeted metabolomics compound lists — the study's conclusions were validated only on five MS-based untargeted metabolomics datasets
- No assay-specific background set can be defined — a non-assay-specific background yields large numbers of false-positive pathways, so conclusions from such a run are unreliable

## Inputs

- Processed metabolomics abundance matrix with group labels (imputed with min value / 2, log2 transformed, features auto-scaled)
- List of differentially abundant metabolites (two-tailed t-test, BH FDR q <= 0.05)
- Assay-specific background/reference set of compounds
- Pathway compound sets from KEGG, Reactome (ChEBI-mapped), or BioCyc (pathways with >= 3 compounds only)

## Outputs

- Per-pathway right-tailed Fisher's exact (hypergeometric) p-values
- List/count of significantly enriched pathways (SEPs) at a chosen threshold (e.g., p <= 0.1 uncorrected)
- SEP loss/gain rates or retained SEP proportions (with SEM) when running parameter-perturbation simulations

## How to apply

Assemble three inputs: the pathway compound sets (keep only pathways with at least 3 compounds), the list of differentially abundant (DA) metabolites derived by two-tailed Student's t-tests with Benjamini-Hochberg FDR adjustment at q <= 0.05, and an assay-specific background set of compounds. Compute a p-value for each pathway using a right-tailed Fisher's exact test based on the hypergeometric distribution (scipy.stats.fisher_exact), calculating p-values only when k (the number of DA metabolites in the pathway) is >= 1. Count significantly enriched pathways (SEPs) at p <= 0.1 without multiple-testing correction when quantifying loss/gain rates. Critically, use an assay-specific background set rather than the full pathway database: a non-assay-specific background produced large numbers of false-positive pathways in the paper's simulations on five MS-based untargeted metabolomics datasets (e.g., MTBLS135, Yachida et al. 2019 stool, MTBLS136). For simulations with randomisation (e.g., background-reduction sweeps), repeat 100 times and report the mean SEP proportion with its standard error of the mean.

## Related tools

- **scipy stats fisher_exact** (Implements the right-tailed (hypergeometric) Fisher's exact test that computes each pathway's ORA p-value)
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, which contains the Python code for all ORA simulations and can be adapted to the user's own data) — github.com/cwieder/metabolomics-ORA
- **pip3** (Installs the notebook dependencies via requirements.txt before launching the ORA simulations) — github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative to run the ORA notebook without local installation) — github.com/cwieder/metabolomics-ORA
- **MetaboAnalyst 4.0** (Converts compound names to KEGG identifiers for mapping metabolites into pathway compound sets)
- **bioservices (Python package)** (Maps KEGG compounds to ChEBI identifiers for Reactome pathway sets)

## Examples

```
pip3 install -r requirements.txt && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Only pathways containing at least 3 compounds are included as ORA input, and p-values are computed only when k >= 1
- SEP counts at p <= 0.1 should be compared between an assay-specific background and a non-assay-specific background; the latter should inflate the SEP count (false positives)
- Randomised simulations must be repeated 100 times and reported as the mean with standard error of the mean
- The DA metabolite list must be derived with BH FDR adjustment at q <= 0.05 before ORA, not from unadjusted p-values
- Results should be reproducible by running the published reproducible_simulations.ipynb notebook (Python 3.8, tested on 16GB RAM)

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, limiting validation of ORA outputs
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs
- Defining the ideal assay-specific background set for a particular dataset remains an open problem
- Two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications
- Study limited to five MS-based untargeted metabolomics datasets

## Evidence

- [intro] P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA.: "Only pathways containing at least 3 compounds were used as input for ORA."
- [abstract] The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways.: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [discussion] Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings.: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings"
