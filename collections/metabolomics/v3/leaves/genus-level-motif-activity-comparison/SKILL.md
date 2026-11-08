---
name: genus-level-motif-activity-comparison
description: Use when a metabolomics peak intensity matrix contains samples grouped
  by taxonomic/genus-level factors (e.g. Rhamnus vs Ziziphus) and you want to rank
  Mass2Motif or Molecular Family metabolite sets by differential activity between
  those groups.
when_to_use_negative:
- Peak features carry no annotations and cannot be grouped into Molecular Families
  or Mass2Motifs — unannotated peaks are excluded from pathway analysis
- The analysis goal requires pathway-level queries against KEGG/Reactome/ChEBI rather
  than user-defined fragmentation-based metabolite sets
- Input is already a ranked activity table rather than a raw peak intensity matrix
  needing imputation, log-2 transformation and standardisation
edam_operation: http://edamontology.org/operation_3081
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PALS
  role: Performs PLAGE decomposition and ranking of user-defined metabolite sets (Molecular
    Families, Mass2Motifs); installed via pip install PALS-pathway
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the intensity and annotation matrices, which can be passed directly
    as DataFrames to DataSource
- name: SciPy
  role: preprocessing module standardises the log-2 transformed intensity matrix to
    zero mean and unit variance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:3a1dc7410f81e881fb0264c385e7aa3f27c3668ec38299ae3c3d0f3e3fa758a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3081
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# genus-level-motif-activity-comparison

## Summary

Applies PALS PLAGE decomposition to user-defined metabolite sets (GNPS Molecular Families and MS2LDA Mass2Motifs) on a peak intensity matrix with genus-level sample factors (e.g. Rhamnus vs Ziziphus), to rank motifs by activity level and p-value and identify which genus overrepresents chemically annotated motif features.

## When to use

Use when a metabolomics peak intensity matrix contains samples grouped by taxonomic/genus-level factors (e.g. Rhamnus vs Ziziphus) and you want to rank Mass2Motif or Molecular Family metabolite sets by differential activity between those groups.

## When NOT to use

- Peak features carry no annotations and cannot be grouped into Molecular Families or Mass2Motifs — unannotated peaks are excluded from pathway analysis
- The analysis goal requires pathway-level queries against KEGG/Reactome/ChEBI rather than user-defined fragmentation-based metabolite sets
- Input is already a ranked activity table rather than a raw peak intensity matrix needing imputation, log-2 transformation and standardisation

## Inputs

- Peak intensity CSV/DataFrame with row_id, sample columns and a group line mapping samples to genus-level factors (e.g. Rhamnus, Ziziphus)
- Peak-to-annotation matrix mapping peak IDs to Mass2Motifs / Molecular Families (or KEGG/ChEBI IDs)
- experimental_design dictionary with 'comparisons' (case/control names) and 'groups' sample mappings

## Outputs

- Ranked pathway/motif activity DataFrame (e.g. rhamnaceae_motif_ranking.csv) with p-value, unq_pw_F, tot_ds_F and F_coverage columns
- Directional activity scores indicating in which genus each motif's features are overrepresented

## How to apply

Load the peak intensity CSV (with a second 'group' line mapping samples to factors such as Rhamnus/Ziziphus) and the peak-to-annotation matrix via pandas. Build an experimental_design dictionary with a case/control comparison (case Rhamnus, control Ziziphus) and group-to-sample mappings, then construct a DataSource with user-defined metabolite sets from GNPS Molecular Families and MS2LDA Mass2Motifs, using min_replace=5000 to impute zero intensities within a factor (all-zero factors take the minimum intensity, partially-zero samples take the mean of non-zero samples). PALS then log-2 transforms and standardises the intensity matrix (zero mean, unit variance) via the Scipy preprocessing module before running PLAGE(ds) and get_pathway_df(). Rank the resulting dataframe by p-value and activity and export, checking that the p-value, unq_pw_F, tot_ds_F and F_coverage columns are present.

## Related tools

- **PALS** (Performs PLAGE decomposition and ranking of user-defined metabolite sets (Molecular Families, Mass2Motifs); installed via pip install PALS-pathway) — github.com/glasgowcompbio/PALS
- **pandas** (Loads the intensity and annotation matrices, which can be passed directly as DataFrames to DataSource)
- **SciPy** (preprocessing module standardises the log-2 transformed intensity matrix to zero mean and unit variance)

## Examples

```
ds = DataSource(int_df, annotation_df, experimental_design, min_replace=5000); df = PLAGE(ds).get_pathway_df(); df.sort_values('p-value').to_csv('rhamnaceae_motif_ranking.csv')
```

## Evaluation signals

- Output CSV exists and contains the required columns: p-value, unq_pw_F, tot_ds_F and F_coverage
- F_coverage equals the proportion of tot_ds_F to unq_pw_F for each metabolite set
- Flavonoid-related motifs (rhamnocitrin, kaempferol, flavonoid core fragment, emodin) rank as differentially expressed with activity higher in Rhamnus
- The cyclopeptidic-alkaloids motif scores higher in Ziziphus, consistent with the known genus-specific chemistry
- Zero intensities within an experimental factor were imputed with min_replace=5000 (minimum) or factor mean as appropriate before transformation

## Limitations

- Peaks without annotations will not be used for pathway analysis
- Multiple peak IDs may map to multiple compound IDs and vice versa, reflecting uncertainty in peak identification
- mPLAGE is more robust but the method still depends on the input intensity matrix; normalisation choices can significantly impact results
- No software version numbers stated and no changelog found for the referenced release

## Evidence

- [other] PALS revealed that Mass2Motifs annotated with flavonoid-related substructures ... are all differently expressed between the Rhamnus and Ziziphus genera: [evidence span withheld — anchor status: not_found]
- [intro] decomposes activity levels in pathways via the PLAGE method: "a complete tool that performs database queries of pathways, decomposes activity levels in pathways via [the PLAGE method]"
- [readme] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value (which can be set by the user)"
- [readme] The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset: "The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage` is the proportion"
- [abstract] mass spectrometry fragmentation patterns grouping via GNPS and MS2LDA: "As an example of an application, PALS is used to analyse metabolites grouped as metabolic pathways and by shared tandem mass spectrometry fragmentation patterns."
