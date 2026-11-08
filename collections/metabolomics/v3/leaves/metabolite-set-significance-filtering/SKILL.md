---
name: metabolite-set-significance-filtering
description: Use when a feature table has metabolites grouped into sets beyond curated
  pathways (GNPS Molecular Families, MS2LDA Mass2Motifs) and the goal is to prioritise
  which sets change significantly between case and control groups.
when_to_use_negative:
- The goal is curated pathway activity over annotated compounds — pathway analysis
  via KEGG/ChEBI/Reactome mapping excludes peaks without annotations, whereas MF grouping
  exists precisely for unknowns
- The question is differential abundance of individual peak features rather than coordinated
  activity of metabolite sets
- Only ORA/GSEA-style enrichment benchmarking against published comparisons is needed
  rather than PLAGE decomposition scoring
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: 'Core library: DataSource construction with imputation/standardisation, PLAGE
    decomposition, and p-value ranking of user-defined metabolite sets'
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the intensity matrix and annotation table as DataFrames passed directly
    into DataSource; writes the filtered ranking CSV
- name: SciPy
  role: Preprocessing module standardises the log2-transformed intensity matrix to
    zero mean and unit variance
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:3f4695b4a791e1b84aff42657aa5feda2e1e710706c50e7eef53348a2ec646b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolite-set-significance-filtering

## Summary

Rank and filter user-defined metabolite sets (e.g. GNPS Molecular Families) by PLAGE activity-level significance, retaining sets with p-value <= 0.05 and a minimum size of 10 molecules using the PALS library.

## When to use

Use when a feature table has metabolites grouped into sets beyond curated pathways (GNPS Molecular Families, MS2LDA Mass2Motifs) and the goal is to prioritise which sets change significantly between case and control groups.

## When NOT to use

- The goal is curated pathway activity over annotated compounds — pathway analysis via KEGG/ChEBI/Reactome mapping excludes peaks without annotations, whereas MF grouping exists precisely for unknowns
- The question is differential abundance of individual peak features rather than coordinated activity of metabolite sets
- Only ORA/GSEA-style enrichment benchmarking against published comparisons is needed rather than PLAGE decomposition scoring

## Inputs

- GNPS intensity matrix CSV (peak features x samples, rows = molecular features)
- Molecular-family annotation table mapping peaks to metabolite sets (GNPS MFs or MS2LDA Mass2Motifs)
- experimental_design dictionary defining case/control groups and comparisons
- min_replace imputation parameter (e.g. 5000)

## Outputs

- Activity-level ranking DataFrame/table with p-values and set sizes per metabolite set
- Filtered CSV of significant sets (p<=0.05, >=10 molecules), e.g. gnps_mf_activity_ranking.csv

## How to apply

Load the GNPS intensity matrix and molecular-family annotation table with pandas, then define an experimental_design dictionary specifying a single case/control comparison (in the American Gut Project example: case = volunteers eating >30 plant-based foods/week, control = <10 plant-based foods/week). Build a PALS DataSource over the user-defined molecular-family metabolite sets with zero-intensity imputation (min_replace=5000), log2 transformation, and SciPy zero-mean/unit-variance standardisation. Run the PLAGE decomposition via PLAGE(ds).get_pathway_df() to compute activity scores and p-values per set, then sort by p-value and filter to p<=0.05 with >=10 molecules per set. The >=10-molecule size cutoff guards against unstable activity scores on tiny families, and the PLAGE decomposition is preferred because mPLAGE is more robust to missing features and noise than ORA or GSEA.

## Related tools

- **PALS** (Core library: DataSource construction with imputation/standardisation, PLAGE decomposition, and p-value ranking of user-defined metabolite sets) — github.com/glasgowcompbio/PALS
- **pandas** (Loads the intensity matrix and annotation table as DataFrames passed directly into DataSource; writes the filtered ranking CSV)
- **SciPy** (Preprocessing module standardises the log2-transformed intensity matrix to zero mean and unit variance)

## Examples

```
from pals.PLAGE import PLAGE; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- On the GNPS American Gut Project data, the filtered ranking recovers exactly 35 significantly changing molecular families (p<=0.05, >=10 molecules)
- The 23-molecule steroid-related molecular family appears among the significant hits with peak intensities higher in the control (<10 plant-based foods/week) group
- Ranking table can be sorted by p-value columns and contains set-size columns consistent with the filtering cutoff
- Imputation applied before scoring: zero intensities within an experimental factor replaced by min_replace (5000) or factor mean, then log2-transformed to zero mean/unit variance

## Limitations

- Activity scores on very small metabolite sets are unstable, hence the >=10-molecule filter; PLAGE/mPLAGE robustness matters most where noise and missing peaks are prevalent
- Imputation replaces all-zero factors with an arbitrary minimum intensity value (min_replace), which can influence downstream scores
- No software version numbers or changelog are stated, so reproducibility pinning must be handled by the practitioner

## Evidence

- [abstract] we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets: "we present PALS (Pathway Activity Level Scoring), a Python library, command line tool, and Web application that performs the ranking of significantly changing metabolite sets"
- [abstract] As further examples, PALS is also applied to human African trypanosomiasis, Rhamnaceae, and American Gut Project data.: "As further examples, PALS is also applied to human African trypanosomiasis, Rhamnaceae, and American Gut Project data."
- [readme] if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [methods] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy"
- [intro] metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed"
