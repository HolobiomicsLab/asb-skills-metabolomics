---
name: gut-metabolomics-differential-analysis
description: Use when a GNPS-derived intensity matrix plus molecular-family annotation
  table must be compared between two sample groups (case vs control, e.g. >30 vs <10
  plant-based foods/week) to find significantly changing metabolite sets at p<=0.05.
when_to_use_negative:
- The goal is classical pathway analysis with annotated compounds mapped to KEGG/ChEBI
  IDs — use PALS' database-driven mode (Reactome/PiMP_KEGG/ChEBI) instead of user-defined
  molecular-family sets
- There is no case/control design — the PLAGE decomposition here computes activity
  scores from explicit case vs control comparisons
- Input is not GNPS/MS2LDA-fragmentation-grouped metabolite sets (molecular families
  or Mass2Motifs) and grouping is unavailable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: Python library providing DataSource (imputation, log-2 transformation, standardisation)
    and PLAGE decomposition to rank molecular-family activity levels
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the GNPS intensity matrix and molecular-family annotation table as DataFrames
    passed directly to the PALS DataSource
- name: SciPy
  role: Preprocessing module standardises the intensity matrix to zero mean and unit
    variance across samples before PLAGE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:ad1f12cbbd7aa6cbc517d226d0407a010c58204bb896c1ffec118c121a11a3de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# gut-metabolomics-differential-analysis

## Summary

Differentially analyse GNPS molecular families from gut-metabolomics data (e.g. the American Gut Project plant-based-foods case/control comparison) by ranking metabolite-set activity levels with the PALS PLAGE decomposition in Python, filtering to significantly changing families.

## When to use

Use when a GNPS-derived intensity matrix plus molecular-family annotation table must be compared between two sample groups (case vs control, e.g. >30 vs <10 plant-based foods/week) to find significantly changing metabolite sets at p<=0.05.

## When NOT to use

- The goal is classical pathway analysis with annotated compounds mapped to KEGG/ChEBI IDs — use PALS' database-driven mode (Reactome/PiMP_KEGG/ChEBI) instead of user-defined molecular-family sets
- There is no case/control design — the PLAGE decomposition here computes activity scores from explicit case vs control comparisons
- Input is not GNPS/MS2LDA-fragmentation-grouped metabolite sets (molecular families or Mass2Motifs) and grouping is unavailable

## Inputs

- GNPS American Gut Project peak-intensity matrix (rows = peak features, columns = samples, CSV)
- Molecular-family annotation table mapping peak IDs to GNPS molecular families (pandas DataFrame)
- experimental_design dictionary specifying groups and case/control comparisons

## Outputs

- Ranked molecular-family activity table with PLAGE activity scores and p-values per case/control comparison
- Full ranking saved as gnps_mf_activity_ranking.csv

## How to apply

Load the GNPS American Gut Project intensity matrix and molecular-family annotation table with pandas, and define an experimental_design dictionary with a single case/control comparison (case = >30 plant-based foods/week, control = <10 plant-based foods/week). Build a PALS DataSource over the user-defined molecular-family metabolite sets with zero-intensity imputation (min_replace=5000), after which the intensity matrix is log-2 transformed and standardised with SciPy's preprocessing module to zero mean and unit variance. Run PLAGE(ds).get_pathway_df() to compute activity-level scores and p-values for each molecular family. Sort the ranking by p-value and retain families with p<=0.05 that contain >=10 molecules; a correct run recovers 35 significant molecular families, including a 23-molecule steroid-related family with peaks higher in the control group. Save the full ranking as gnps_mf_activity_ranking.csv.

## Related tools

- **PALS** (Python library providing DataSource (imputation, log-2 transformation, standardisation) and PLAGE decomposition to rank molecular-family activity levels) — github.com/glasgowcompbio/PALS
- **pandas** (Loads the GNPS intensity matrix and molecular-family annotation table as DataFrames passed directly to the PALS DataSource)
- **SciPy** (Preprocessing module standardises the intensity matrix to zero mean and unit variance across samples before PLAGE)

## Examples

```
from pals.PLAGE import PLAGE; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, database_name, min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- Ranking filtered to p<=0.05 and >=10 molecules yields exactly 35 significantly changing molecular families
- The 23-molecule steroid-related molecular family appears among the significant hits with peak intensities higher in the control group (<10 plant-based foods/week)
- Standardised intensity matrix has zero mean and unit variance across samples (verifiable via scipy.stats.zscore output)
- Zero-intensity features are imputed with min_replace=5000 before transformation
- Output file gnps_mf_activity_ranking.csv contains p-value columns that can be sorted as in README output guidance

## Limitations

- Normalisation choices can significantly impact pathway-analysis results; PALS is explicitly framed as a framework to investigate this, so results depend on the min_replace and standardisation settings
- No software version numbers are stated in the article or README, and no changelog is available, complicating reproducibility pinning
- PLAGE is only one ranking method; ORA and GSEA are also included in PALS and may give different robustness behaviour on noisy or peak-missing metabolomics data

## Evidence

- [methods] Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [abstract] we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets: "we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets"
- [readme] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy"
- [intro] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed. This includes in particular *Molecular Families* from"
