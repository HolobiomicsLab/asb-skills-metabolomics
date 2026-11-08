---
name: plage-metabolite-set-decomposition
description: Use when a peak/feature intensity matrix with case vs control groups
  must be ranked at the metabolite-set level (pathways or fragmentation-derived sets
  such as GNPS Molecular Families), and univariate per-peak testing is insufficient.
when_to_use_negative:
- When the goal is per-peak differential abundance testing rather than ranking of
  metabolite sets/pathways
- When peaks cannot be annotated or grouped into metabolite sets — unannotated peaks
  are excluded from pathway analysis, so results would be empty or misleading
- When the input is already a completed pathway-ranking table rather than raw peak
  intensities
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: PALS
  role: 'Core library: DataSource construction, zero-intensity imputation, PLAGE decomposition
    and pathway/MF ranking (pip install PALS-pathway)'
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loading the intensity matrix and annotation table; pandas DataFrames are passed
    directly to DataSource in programmatic use
- name: SciPy
  role: Preprocessing module used to standardise the intensity matrix to zero mean
    and unit variance across samples after log2 transformation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:9167235acbd6bcfbe1196d49a68242d9c0504e788df3747a42fe21078eaff075
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# plage-metabolite-set-decomposition

## Summary

Rank metabolite sets — Reactome pathways, GNPS Molecular Families, or MS2LDA Mass2Motifs — by their case/control activity change using the PALS library's PLAGE decomposition, which is reported to be more robust to missing peaks and noise than ORA or GSEA.

## When to use

Use when a peak/feature intensity matrix with case vs control groups must be ranked at the metabolite-set level (pathways or fragmentation-derived sets such as GNPS Molecular Families), and univariate per-peak testing is insufficient.

## When NOT to use

- When the goal is per-peak differential abundance testing rather than ranking of metabolite sets/pathways
- When peaks cannot be annotated or grouped into metabolite sets — unannotated peaks are excluded from pathway analysis, so results would be empty or misleading
- When the input is already a completed pathway-ranking table rather than raw peak intensities

## Inputs

- Peak-feature intensity matrix (CSV; rows = peak features, columns = samples, optional second-line group labels)
- Annotation table mapping peak IDs to compound/database entities (or GNPS molecular-family membership for user-defined metabolite sets)
- experimental_design dictionary with 'comparisons' listing case/control group pairs (e.g. {'case': 'beer1', 'control': 'beer2', 'name': 'beer1/beer2'})
- min_replace imputation value (default 5000)

## Outputs

- Metabolite-set activity ranking DataFrame with PLAGE activity scores and p-values per set
- Filtered table of significantly changing metabolite sets (p<=0.05, e.g. >=10 molecules for MFs), saved as e.g. gnps_mf_activity_ranking.csv

## How to apply

Load the intensity matrix and annotation table (e.g. the GNPS American Gut Project files in the PALS GNPS_analysis example) with pandas, and define an experimental_design dictionary mapping samples to case (>30 plant-based foods/week) and control (<10 plant-based foods/week) plus a single case/control comparison. Build a PALS DataSource over the user-defined metabolite sets with min_replace=5000 so that all-zero intensities within a factor are imputed to the minimum intensity value and partially-zero values to the mean of non-zero samples in that factor; the matrix is then log2-transformed and SciPy-standardised to zero mean and unit variance. Instantiate PLAGE(ds) and call get_pathway_df() to obtain per-set activity scores and p-values for the case/control comparison (PLAGE, ORA and GSEA are all available; PLAGE is preferred because the decomposition approach handles any metabolite-set grouping). Sort the ranking by p-value and filter to significant sets — for the American Gut molecular-family analysis this means p<=0.05 and >=10 molecules per family. Save the ranking (e.g. gnps_mf_activity_ranking.csv) and check invariants against the published result.

## Related tools

- **PALS** (Core library: DataSource construction, zero-intensity imputation, PLAGE decomposition and pathway/MF ranking (pip install PALS-pathway)) — github.com/glasgowcompbio/PALS
- **pandas** (Loading the intensity matrix and annotation table; pandas DataFrames are passed directly to DataSource in programmatic use)
- **SciPy** (Preprocessing module used to standardise the intensity matrix to zero mean and unit variance across samples after log2 transformation)

## Examples

```
from pals.PLAGE import PLAGE; from pals.feature_extraction import DataSource; ds = DataSource(int_df, annotation_df, experimental_design, database_name, min_replace=5000); df = PLAGE(ds).get_pathway_df()
```

## Evaluation signals

- On the GNPS American Gut Project data, the analysis must recover 35 significantly changing molecular families (p<=0.05, >=10 molecules)
- The 23-molecule steroid-related molecular family must appear among significant hits with peak intensities higher in the control group (<10 plant-based foods/week)
- After preprocessing, the standardised intensity matrix must have zero mean and unit variance across samples
- Zero intensities must be imputed: all-zero factors replaced by min_replace (default 5000), partially-zero factors by the mean of non-zero samples
- Ranking table must be sortable by p-value columns and include coverage columns (unq_pw_F, tot_ds_F, F_coverage) for pathway queries

## Limitations

- Peaks without annotations are not used for pathway analysis, so poorly annotated datasets will under-cover pathways
- Normalisation choices can significantly impact pathway analysis results; PALS offers a framework to investigate this but results are normalisation-dependent
- Offline Reactome mode only provides metabolic pathways; all-pathway queries need an online Neo4j Reactome server
- No software version numbers or changelog are stated, complicating reproducibility pinning

## Evidence

- [intro] decomposes activity levels in pathways via the PLAGE method: "Here we introduce **PALS (Pathway Activity Level Scoring)**, a complete tool that performs database queries of pathways, decomposes activity levels in pathways via [the PLAGE method]"
- [abstract] mPLAGE robustness versus ORA/GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] README: preprocessing pipeline: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [readme] README: min_replace parameter: "--min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
- [readme] README: imputation rule: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value (which can be set by the user)"
