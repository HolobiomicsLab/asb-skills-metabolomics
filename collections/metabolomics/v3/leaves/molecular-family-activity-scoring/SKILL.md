---
name: molecular-family-activity-scoring
description: Use when you have GNPS molecular-family groupings of (possibly unknown)
  metabolites and want to identify whole MFs significantly changing (e.g. p<=0.05,
  >=10 molecules) between defined case/control sample groups.
when_to_use_negative:
- Your peaks have database annotations (KEGG/ChEBI IDs) and you want pathway-level
  rather than fragmentation-grouped analysis — use the standard Reactome/KEGG pathway
  mode of PALS instead
- Your data has no defined case/control comparison (e.g. only one experimental group)
  — PLAGE scoring in PALS is defined in terms of a case vs control design
- You need per-molecule differential analysis rather than activity levels of grouped
  metabolite sets — MF scoring ranks sets, not individual features
tools:
- name: PALS
  role: Python library providing DataSource, PLAGE decomposition and get_pathway_df()
    used to score molecular-family activity levels; installable via `pip install PALS-pathway`
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Used programmatically to load the GNPS intensity matrix and molecular-family
    annotation tables as DataFrames passed directly to the PALS DataSource
- name: SciPy
  role: Its preprocessing module standardises the (log2-transformed) intensity matrix
    to zero mean and unit variance before PLAGE decomposition
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:448d811f494bfc3872253fb449af2c99e3c1844ee4521f3ab9da901f5d693073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# molecular-family-activity-scoring

## Summary

Ranks GNPS Molecular Families (MFs) — groups of metabolites sharing fragmentation patterns — by their differential activity between case and control groups using the PLAGE decomposition implemented in PALS, yielding p-values per molecular family instead of per individual peak.

## When to use

Use when you have GNPS molecular-family groupings of (possibly unknown) metabolites and want to identify whole MFs significantly changing (e.g. p<=0.05, >=10 molecules) between defined case/control sample groups.

## When NOT to use

- Your peaks have database annotations (KEGG/ChEBI IDs) and you want pathway-level rather than fragmentation-grouped analysis — use the standard Reactome/KEGG pathway mode of PALS instead
- Your data has no defined case/control comparison (e.g. only one experimental group) — PLAGE scoring in PALS is defined in terms of a case vs control design
- You need per-molecule differential analysis rather than activity levels of grouped metabolite sets — MF scoring ranks sets, not individual features

## Inputs

- GNPS peak intensity CSV matrix (rows = peak features, columns = samples, optional group row)
- GNPS molecular-family annotation table mapping peak IDs to molecular families (user-defined metabolite sets)
- experimental_design dictionary with 'comparisons' listing case/control pairs and sample-to-group assignments
- min_replace imputation parameter (default 5000)

## Outputs

- Activity ranking dataframe/table of molecular families with PLAGE activity scores, p-values and molecule counts (e.g. gnps_mf_activity_ranking.csv)
- Filtered list of significantly changing molecular families (p<=0.05, >=10 molecules)

## How to apply

Load the GNPS American Gut Project intensity matrix and molecular-family annotation table with pandas, and define an `experimental_design` dictionary with one case/control comparison (e.g. case = >30 plant-based foods/week, control = <10 plant-based foods/week). Build a PALS `DataSource` over the user-defined molecular-family metabolite sets with zero-intensity imputation (`min_replace=5000`: all-zero samples in a factor get the minimum intensity, partially-zero get the factor mean), then log2-transform and standardise with SciPy's preprocessing module to zero mean and unit variance. Run the PLAGE decomposition via `PLAGE(ds).get_pathway_df()` to get activity scores and p-values per molecular family for the case/control comparison. Sort the ranking by p-value and filter to significant families (p<=0.05) with a minimum size threshold such as >=10 molecules. The rationale is that the decomposition approach works on any group of metabolite sets, not just curated pathways, so unknown metabolites grouped by GNPS fragmentation spectra can be prioritised; validation requires reproducing the expected hit count and specific families (e.g. the 23-molecule steroid-related MF with elevated control-group peaks).

## Related tools

- **PALS** (Python library providing DataSource, PLAGE decomposition and get_pathway_df() used to score molecular-family activity levels; installable via `pip install PALS-pathway`) — https://github.com/glasgowcompbio/PALS
- **pandas** (Used programmatically to load the GNPS intensity matrix and molecular-family annotation tables as DataFrames passed directly to the PALS DataSource)
- **SciPy** (Its preprocessing module standardises the (log2-transformed) intensity matrix to zero mean and unit variance before PLAGE decomposition)

## Examples

```
from pals.PLAGE import PLAGE; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- Reproduce the reference result: 35 significantly changing molecular families (p<=0.05, >=10 molecules) between the >30 and <10 plant-based-foods/week groups in the GNPS American Gut Project data
- Confirm the 23-molecule steroid-related molecular family appears among the significant hits with peak intensities higher in the control group
- Check the intensity matrix was imputed with min_replace=5000, log2-transformed, and standardised to zero mean and unit variance before scoring
- Verify the ranking table is sortable by p-value columns and contains molecule-count columns sufficient to apply the >=10 molecule filter
- Confirm the saved output (gnps_mf_activity_ranking.csv) matches the full unfiltered ranking, with filtering applied downstream

## Limitations

- The article/README state no software version numbers and no changelog is available, so reproducibility depends on pinning the repository commit yourself
- Imputation only addresses zeros (min-value replacement for all-zero factors, mean replacement for partial zeros); other missing-value patterns are not handled
- Robustness to noise and missing peaks is claimed relative to ORA and GSEA, but heavily degraded MF peak data may still affect PLAGE activity estimates
- pandas dataframes must be correctly shaped and the experimental_design dictionary correctly initialised — mis-specified groups/comparisons silently change the scored contrasts

## Evidence

- [abstract] we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets: "we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets"
- [methods] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy"
- [methods] analysing potentially unknown metabolites that have been grouped into Molecular Families (by GNPS) and into Mass2Motifs (by MS2LDA): "analysing potentially unknown metabolites that have been grouped into Molecular Families (by GNPS)"
- [readme] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed. This includes in particular *Molecular Families* from [GNPS]: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed"
- [readme] if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value (which can be set by the user): "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
