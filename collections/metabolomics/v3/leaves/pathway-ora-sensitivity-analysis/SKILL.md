---
name: pathway-ora-sensitivity-analysis
description: Use when interpreting metabolomics data with ORA and you need to know
  how sensitive the significant-pathway result is to the background set, the DA-metabolite
  list construction (e.g. BH FDR q <= 0.05 vs p-value ordering), or the pathway database
  (KEGG, Reactome, BioCyc).
when_to_use_negative:
- Input is already a validated ground-truth set of enriched pathways — the authors
  note this study is limited by the lack of availability of a ground-truth dataset
  where the identities of enriched pathways are known
- You need a definitive single answer for which pathway database is best — an extensive
  comparison of pathway databases is beyond the scope of this work and readers are
  referred to other studies
- Your data come from a platform without a chromatographic separation step (e.g. Quiros
  et al. 2017, Fuhrer et al. 2017) and metabolite identifications carry high uncertainty,
  since the sensitivity conclusions assume reasonably confident compound assignments
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Python 3.8
  role: Implementation language for all simulations and ORA computation (tested on
    MacOS v11.2.3, 16GB RAM)
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: 'Custom ORA: right-tailed Fisher''s exact test to calculate pathway p-values'
  repo: https://github.com/cwieder/metabolomics-ORA
- name: MetaboAnalyst 4.0
  role: Conversion of compound names to KEGG identifiers
- name: bioservices (Python package) v 1.7.1
  role: Mapping KEGG compounds to ChEBI identifiers for the Reactome pathway set
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb that generates all results; adaptable
    to the user's own data
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Alternative remote execution of the same notebook without local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:4d275081a506982b50b94710da424ee3c2f2d7ec34ca4bf38480ed856d39d1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - MetaboAnalyst 4.0
  - Python
  - Python 3.8
  - bioservices (Python package) v 1.7.1
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# pathway-ora-sensitivity-analysis

## Summary

Systematically varies over-representation analysis (ORA) inputs — background set, differential metabolite selection threshold, pathway database, misidentification rate, and assay-specific mapping — to quantify how each choice changes the list of significantly enriched pathways (SEPs), using five MS-based untargeted metabolomics datasets and right-tailed Fisher's exact (hypergeometric) tests in Python.

## When to use

Use when interpreting metabolomics data with ORA and you need to know how sensitive the significant-pathway result is to the background set, the DA-metabolite list construction (e.g. BH FDR q <= 0.05 vs p-value ordering), or the pathway database (KEGG, Reactome, BioCyc).

## When NOT to use

- Input is already a validated ground-truth set of enriched pathways — the authors note this study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known
- You need a definitive single answer for which pathway database is best — an extensive comparison of pathway databases is beyond the scope of this work and readers are referred to other studies
- Your data come from a platform without a chromatographic separation step (e.g. Quiros et al. 2017, Fuhrer et al. 2017) and metabolite identifications carry high uncertainty, since the sensitivity conclusions assume reasonably confident compound assignments

## Inputs

- Processed metabolomics abundance matrix (metabolites × samples) from five publicly available datasets, e.g. MTBLS135 (MetaboLights), MTBLS136 (MetaboLights), Yachida et al. 2019 colorectal cancer stool dataset, Quiros et al. 2017, Fuhrer et al. 2017
- Condition/group labels for each sample (e.g. 12 Hi-Myc genotype vs 12 wild-type Mus musculus tissue samples; 149 healthy controls vs 148 colorectal cancer stool samples; 667 controls vs 332 estrogen users)
- Pathway sets from KEGG, Reactome, and BioCyc databases (pathways restricted to those containing at least 3 compounds)
- Compound identifier mapping files (dataset metabolite IDs to KEGG, ChEBI, and BioCyc IDs)
- Candidate list of metabolites of interest (DA metabolites) selected via t-test p-value thresholds

## Outputs

- Lists of significantly enriched pathways (SEPs) for each ORA parameter combination
- Pathway loss and gain rates (proportion of original SEPs no longer significant / newly significant at p <= 0.1 under each perturbation)
- Mean and standard error of the mean across 100 randomised simulation repeats
- Recommendations and minimal reporting guidelines for ORA in metabolomics

## How to apply

Post-process each abundance matrix (impute missing values as half of each metabolite's minimum, log2-transform, then auto-scale) and convert metabolite identifiers to KEGG, Reactome (ChEBI), and BioCyc identifiers via MetaboAnalyst 4.0 ID conversion and the Python bioservices v1.7.1 package. Select differentially abundant metabolites by two-tailed Student's t-tests with Benjamini-Hochberg FDR (or Bonferroni) adjustment at q <= 0.05, then run right-tailed Fisher's exact ORA using scipy.stats.fisher_exact, restricted to pathways containing at least 3 compounds, computing a pathway p-value only when k >= 1 DA metabolite maps to the pathway. To probe robustness, re-run the analysis while systematically varying: the background set (non-assay-specific vs assay-specific), the DA-list threshold (0.005, 0.05, 0.1), the pathway database (KEGG, Reactome, BioCyc), and a simulated metabolite misidentification rate f% in which compounds are swapped with isobaric (molecular weight within ±20 ppm) or identical-chemical-formula counterparts present in the KEGG pathway set. Simulations with randomisation are repeated 100 times and reported as mean plus standard error of the mean; SEPs are counted at p <= 0.1 without multiple testing correction so losses and gains can be attributed to each parameter change.

## Related tools

- **Python 3.8** (Implementation language for all simulations and ORA computation (tested on MacOS v11.2.3, 16GB RAM)) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Custom ORA: right-tailed Fisher's exact test to calculate pathway p-values) — https://github.com/cwieder/metabolomics-ORA
- **MetaboAnalyst 4.0** (Conversion of compound names to KEGG identifiers)
- **bioservices (Python package) v 1.7.1** (Mapping KEGG compounds to ChEBI identifiers for the Reactome pathway set)
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb that generates all results; adaptable to the user's own data) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Alternative remote execution of the same notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Invariant check: with a non-assay-specific background set the result must show large numbers of false-positive pathways compared to an assay-specific background set
- Invariant check: across KEGG, Reactome, and BioCyc, SEP counts must differ markedly in both number and function for the same input data
- Invariant check: simulated misidentification as low as 4% must produce both gain of false-positive pathways and loss of truly significant pathways across all datasets
- Invariant check: randomised simulations run 100 times must report mean plus standard error of the mean, not a single run
- Sanity check: ORA p-values come from a right-tailed Fisher's exact test (scipy.stats.fisher_exact) and only pathways with >= 3 compounds are used

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so 'correct' SEPs cannot be externally validated
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs, which inflates the number of significant pathways
- Defining the ideal assay-specific background set remains an open problem
- No benchmark level of metabolite misidentification typical of metabolomics studies exists, so the simulated misidentification rates cannot be anchored to real-world error rates
- Two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, so metabolite identifications there carry higher uncertainty
- Study limited to five MS-based untargeted metabolomics datasets; conclusions may not generalise to other platforms
- Simulated ground-truth datasets would be too simplistic, while experimental ground-truth datasets cannot pinpoint the exact pathway(s) affected by a perturbation

## Evidence

- [abstract] changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
- [abstract] The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways.: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [abstract] Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets.: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
- [abstract] Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results in both the number and function of: "Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results in both the number and function of"
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: [evidence span withheld — anchor status: not_found]
- [discussion] The list of metabolites of interest was determined using a series of two-tailed student's t-tests ... P-values were adjusted using the Benjamini-Hochberg False discovery rate (BH FDR) procedure: [evidence span withheld — anchor status: not_found]
- [discussion] we randomly selected f % (f = 0, 1, ...X%) of compounds that had at least one other compound with a molecular weight within ±20ppm (approximately isobaric compound) present in the KEGG pathway set: [evidence span withheld — anchor status: not_found]
- [discussion] Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean.: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [discussion] This study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known.: "This study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known."
- [discussion] The pathway loss rate represents the proportion of the original pathways (0% misidentification) significant at p <= 0.1 that are no longer significant at f % misidentification.: [evidence span withheld — anchor status: not_found]
- [discussion] All simulations were performed using Python (v 3.8).: "All simulations were performed using Python (v 3.8)."
- [discussion] Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled: "Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled"
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [readme] the Jupyter notebook can now be run as an analogous Google <a href="https://research.google.com/colaboratory/faq.html">Colab</a> version: "the Jupyter notebook can now be run as an analogous Google <a href="https://research.google.com/colaboratory/faq.html">Colab</a> version"
