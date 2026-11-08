---
name: molecular-family-metabolite-set-analysis
description: Use when peak intensity data from an untargeted metabolomics experiment
  must be ranked as grouped metabolite sets (Molecular Families, Mass2Motifs) by differential
  activity between experimental factors, e.g. Rhamnus vs Ziziphus.
when_to_use_negative:
- The task is single-peak differential testing rather than ranking metabolite sets
  as a whole
- Peaks cannot be assigned to any metabolite set (no Molecular Family/Mass2Motif grouping
  or compound annotations), since unannotated peaks are not used for pathway analysis
- The goal is Reactome pathway queries for transcriptomics/proteomics data (UniProt/ENSEMBL
  databases) rather than metabolite-set ranking from fragmentation-spectrum groupings
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PALS
  role: Python library performing the PLAGE decomposition and metabolite-set ranking;
    installed via pip install PALS-pathway or Git
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Loads the intensity and annotation matrices as dataframes passed directly
    to DataSource
- name: SciPy
  role: preprocessing module used to standardise the log-2 intensity matrix to zero
    mean and unit variance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:ade24c4fd071d15ad8cc492bd1b92833d0f6494f96e08b69d8fe2d6a669b7ce3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# molecular-family-metabolite-set-analysis

## Summary

Ranks user-defined metabolite sets — GNPS Molecular Families and MS2LDA Mass2Motifs — by activity level using the PALS PLAGE decomposition on LC-MS peak intensity matrices, e.g. detecting that flavonoid-related Mass2Motifs (rhamnocitrin, kaempferol, flavonoid core, emodin) are overrepresented in Rhamnus versus Ziziphus.

## When to use

Use when peak intensity data from an untargeted metabolomics experiment must be ranked as grouped metabolite sets (Molecular Families, Mass2Motifs) by differential activity between experimental factors, e.g. Rhamnus vs Ziziphus.

## When NOT to use

- The task is single-peak differential testing rather than ranking metabolite sets as a whole
- Peaks cannot be assigned to any metabolite set (no Molecular Family/Mass2Motif grouping or compound annotations), since unannotated peaks are not used for pathway analysis
- The goal is Reactome pathway queries for transcriptomics/proteomics data (UniProt/ENSEMBL databases) rather than metabolite-set ranking from fragmentation-spectrum groupings

## Inputs

- Peak intensity CSV matrix (rows = peak features, columns = samples, second line with group/factor labels)
- Peak-to-annotation CSV matrix (peak ID mapped to metabolite/annotation ID)
- experimental_design dictionary with groups and case/control comparisons
- User-defined metabolite sets: GNPS Molecular Families and MS2LDA Mass2Motifs annotations

## Outputs

- Ranked pathway/motif dataframe written to CSV (e.g. rhamnaceae_motif_ranking.csv) with p-value, unq_pw_F, tot_ds_F and F_coverage columns
- Per-metabolite-set activity level scores from the PLAGE decomposition

## How to apply

Load the peak intensity matrix (second line carrying sample group labels, e.g. Rhamnus/Ziziphus factors) and the peak-to-annotation matrix with pandas, then build an experimental_design dictionary listing case/control comparisons (e.g. case Rhamnus, control Ziziphus) plus group-to-sample mappings. Construct a DataSource with user-defined metabolite sets derived from GNPS Molecular Families and MS2LDA Mass2Motifs, applying the default min_replace=5000 imputation: all-zero intensities within a factor are replaced by the minimum value, partially-zero factors by the mean of non-zero values. Let PALS transform the matrix to log-2 base and standardise it via the Scipy preprocessing module to zero mean and unit variance. Run the PLAGE decomposition (PLAGE(ds) followed by get_pathway_df()) to compute activity scores per Molecular Family and Mass2Motif, then rank motifs by p-value and activity and export the ranking (columns p-value, unq_pw_F, tot_ds_F, F_coverage) e.g. as rhamnaceae_motif_ranking.csv.

## Related tools

- **PALS** (Python library performing the PLAGE decomposition and metabolite-set ranking; installed via pip install PALS-pathway or Git) — https://github.com/glasgowcompbio/PALS
- **pandas** (Loads the intensity and annotation matrices as dataframes passed directly to DataSource)
- **SciPy** (preprocessing module used to standardise the log-2 intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control --min_replace 5000
```

## Evaluation signals

- Output CSV exists and contains the p-value, unq_pw_F, tot_ds_F and F_coverage columns
- Ranked activities show flavonoid-related motifs (rhamnocitrin, kaempferol, flavonoid core fragment, emodin) scoring higher in Rhamnus while the cyclopeptidic-alkaloids motif scores higher in Ziziphus
- Imputation invariants hold: all-zero factors replaced with min_replace=5000, partial zeros replaced by factor means
- Standardised intensity matrix has zero mean and unit variance across samples after log-2 transformation
- Only peaks with annotations participate in the set-level activity scoring

## Limitations

- Normalisation choices can have a significant impact on pathway analysis results; PALS offers a framework to investigate this but the default min_replace=5000 imputation is a user choice
- Unannotated peaks are excluded from pathway analysis, so coverage of the intensity matrix depends on annotation completeness
- Offline Reactome mode only provides metabolic pathways for a limited set of species; Mass2Motif/Molecular Family analysis relies on user-defined sets instead

## Evidence

- [methods] Data imputation is performed to the intensity matrix when it is loaded: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [methods] Log-2 transform and standardisation: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy"
- [readme] Output ranking columns: "The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage`"
- [intro] Fragmentation-spectrum metabolite sets are valid PALS inputs: "metabolite sets obtained from the grouping of metabolites according to their fragmentation spectra can also be analysed. This includes in particular *Molecular Families* from"
- [abstract] mPLAGE robustness compared to ORA and GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
