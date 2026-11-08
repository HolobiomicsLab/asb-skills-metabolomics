---
name: case-control-pathway-ranking-analysis
description: Use when you have a peak-intensity matrix with case/control experimental
  groups (e.g. Stage_1/Control, Stage_2/Control comparisons) plus peak-to-compound
  annotations (KEGG or ChEBI IDs) and need to rank significantly changing metabolite
  sets.
when_to_use_negative:
- When peak features lack compound annotations — peaks without annotations will not
  be used for pathway analysis
- When the goal is benchmarking against ORA/GSEA rather than the more robust mPLAGE
  decomposition — although PALS includes ORA and GSEA, the paper reports mPLAGE is
  more robust to noise and missing peaks
- When analysing groups of metabolites that are not database pathways without supplying
  the user-defined metabolite sets (e.g. GNPS Molecular Families or MS2LDA Mass2Motifs)
  — these must be provided explicitly
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: Python library, command-line tool and web application that performs pathway
    database queries and ranks significantly changing metabolite sets by decomposing
    pathway activity levels via the PLAGE method
  repo: github.com/glasgowcompbio/PALS
- name: PALS Viewer
  role: Streamlit-based web interface to run PALS, analyse pathway ranking results
    and inspect significantly changing pathways, including prioritisation of GNPS
    Molecular Families and MS2LDA Mass2Motifs
- name: pandas
  role: Provides int_df and annotation_df DataFrames passed directly when PALS is
    used programmatically
- name: SciPy
  role: Preprocessing module used to standardise the log-2 transformed intensity matrix
    to zero mean and unit variance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:362ee6ab187be0212fa2a38927eed22c48d6d60bea37dcda71d90855bac48bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - PALS Viewer
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# case-control-pathway-ranking-analysis

## Summary

Rank metabolic pathways by their activity-level changes between case and control groups using PALS (Pathway Activity Level Scoring), which decomposes pathway activity via the mPLAGE method on metabolomics peak-intensity data and is more robust to noise and missing peaks than ORA or GSEA.

## When to use

Use when you have a peak-intensity matrix with case/control experimental groups (e.g. Stage_1/Control, Stage_2/Control comparisons) plus peak-to-compound annotations (KEGG or ChEBI IDs) and need to rank significantly changing metabolite sets.

## When NOT to use

- When peak features lack compound annotations — peaks without annotations will not be used for pathway analysis
- When the goal is benchmarking against ORA/GSEA rather than the more robust mPLAGE decomposition — although PALS includes ORA and GSEA, the paper reports mPLAGE is more robust to noise and missing peaks
- When analysing groups of metabolites that are not database pathways without supplying the user-defined metabolite sets (e.g. GNPS Molecular Families or MS2LDA Mass2Motifs) — these must be provided explicitly

## Inputs

- Peak-intensity CSV matrix (rows = peak features with peak id in column one, columns = samples; optional second line declaring sample group membership, e.g. beer1/beer2/...)
- Annotation CSV matrix with two columns: peak ID (row_id) and assigned compound annotation as KEGG (e.g. C00148) or ChEBI IDs
- Comparison specification of case/control pairs (e.g. Stage_1/Control Stage_2/Control)
- Pathway database choice: PiMP_KEGG, COMPOUND, ChEBI (metabolomics) or UniProt, ENSEMBL (proteomics/transcriptomics); optional species for Reactome queries

## Outputs

- Pathway ranking CSV (test_output.csv) with p-value columns, sorted by p-value
- Per-pathway coverage metrics: unq_pw_F (unique formulae), tot_ds_F (formula hits in dataset), F_coverage (tot_ds_F/unq_pw_F)
- Per-comparison pathway activity-level scores for significantly changing pathways

## How to apply

Install PALS via `pip install PALS-pathway` and run from the command line with pals/run.py, choosing PLAGE (or ORA/GSEA for benchmarking), an intensity CSV, an annotation CSV, an output file, a pathway database (`--db PiMP_KEGG` for the KEGG compound database exported from PiMP), and comparisons such as `--comparisons Stage_1/Control Stage_2/Control`. On load, PALS imputes the intensity matrix: all-zero values in an experimental factor are replaced by the minimum intensity (`--min_replace`, default 5000) and partially-zero factors by the mean of non-zero samples in that factor; the data is then log-2 transformed and standardised with Scipy to zero mean and unit variance. Peaks lacking annotations are excluded from pathway analysis, and multiple peak-to-compound mappings are allowed. Sort the output by the p-value columns and interpret pathway coverage using `unq_pw_F` (unique formulae per pathway), `tot_ds_F` (formula hits in the dataset), and `F_coverage` (tot_ds_F/unq_pw_F). In the HAT case study this reproduced the paper's counts of significantly changing pathways (52/74 Plasma S1:C, S2:C; 42/128 and 42/143 CSF) and confirmed decreasing CSF activity of T. brucei auxotrophic amino-acid pathways in stage 2 disease.

## Related tools

- **PALS** (Python library, command-line tool and web application that performs pathway database queries and ranks significantly changing metabolite sets by decomposing pathway activity levels via the PLAGE method) — github.com/glasgowcompbio/PALS
- **PALS Viewer** (Streamlit-based web interface to run PALS, analyse pathway ranking results and inspect significantly changing pathways, including prioritisation of GNPS Molecular Families and MS2LDA Mass2Motifs)
- **pandas** (Provides int_df and annotation_df DataFrames passed directly when PALS is used programmatically)
- **SciPy** (Preprocessing module used to standardise the log-2 transformed intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Output CSV must contain p-value columns plus unq_pw_F, tot_ds_F and F_coverage (tot_ds_F/unq_pw_F) columns
- For the HAT dataset, counts of significantly changing pathways must match the paper's reported values: 52/74 (Plasma S1:C, S2:C) and 42/128, 42/143 (CSF)
- CSF S2-vs-control ranking must show decreasing pathway activity for T. brucei auxotrophic amino acids, with glycine excluded as undetected (Supplementary Table S6 direction check)
- Imputation invariants: all-zero values within a factor replaced by --min_replace (default 5000); partial zeros replaced by the factor's non-zero mean; matrix standardised to zero mean and unit variance
- Peaks without KEGG/ChEBI annotations must be excluded from the pathway analysis

## Limitations

- Normalisation can have a significant impact on pathway analysis results, and PALS offers a framework to investigate this — results depend on the imputation and standardisation choices
- Peaks that do not have annotations will not be used for pathway analysis, so poorly annotated metabolomics datasets lose coverage
- Offline Reactome mode only provides metabolic pathways for a fixed set of species unless --use_all_reactome_pathways and --connect_to_reactome_server are used
- Uncertainty in peak identification means multiple peak IDs may map to multiple compound IDs and vice versa

## Evidence

- [abstract] mPLAGE is more robust to missing features and noisy data than ORA and GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] imputation and standardisation procedure: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] Scipy standardisation: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [readme] output column semantics: "The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage` is the proportion"
- [methods] unannotated peaks excluded: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
