---
name: trypanosome-amino-acid-auxotrophy-interpretation
description: Use when interpreting PALS/PLAGE pathway rankings from CSF metabolomics
  of HAT patients, to verify that decreasing T. brucei auxotrophic amino acid pathway
  activities match Supplementary Table S6 directions.
when_to_use_negative:
- The peak features lack compound annotations to KEGG/ChEBI IDs — unannotated peaks
  are not used for pathway analysis, so auxotrophic amino acid pathways cannot be
  ranked
- The dataset is not CSF/plasma metabolomics with Control, Stage_1 and Stage_2 HAT
  groups — the auxotrophic-amino-acid interpretation and Table S6 comparison are specific
  to this case study
- The research question concerns ORA or GSEA ranking rather than PLAGE decomposition
  of pathway activity levels
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: PALS
  role: Runs PLAGE decomposition and produces the pathway ranking CSV whose CSF S2-vs-control
    output is checked for decreasing T. brucei auxotrophic amino acid pathway activities;
    installed via `pip install PALS-pathway`
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Stores and passes the intensity (int_df) and annotation (annot_df) dataframes
    when PALS is used programmatically
- name: SciPy
  role: Standardises the imputed and log-2 transformed intensity matrix to zero mean
    and unit variance via its preprocessing module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:370523e17e0be266e1e68f2b7b7bb3a5ee2883405f29eccd1b22e8b0514550b4
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# trypanosome-amino-acid-auxotrophy-interpretation

## Summary

A biological-interpretation skill that validates pathway-level metabolomics findings by checking whether amino acids required by Trypanosoma brucei (auxotrophic amino acids) decrease in CSF of stage 2 human African trypanosomiasis patients, as reproduced with the PALS pipeline.

## When to use

Use when interpreting PALS/PLAGE pathway rankings from CSF metabolomics of HAT patients, to verify that decreasing T. brucei auxotrophic amino acid pathway activities match Supplementary Table S6 directions.

## When NOT to use

- The peak features lack compound annotations to KEGG/ChEBI IDs — unannotated peaks are not used for pathway analysis, so auxotrophic amino acid pathways cannot be ranked
- The dataset is not CSF/plasma metabolomics with Control, Stage_1 and Stage_2 HAT groups — the auxotrophic-amino-acid interpretation and Table S6 comparison are specific to this case study
- The research question concerns ORA or GSEA ranking rather than PLAGE decomposition of pathway activity levels

## Inputs

- Peak-intensity CSV matrix (rows = peak features, first column peak id, second line group labels, e.g. notebooks/test_data/HAT/int_df.csv)
- Compound-annotation CSV matrix mapping peak IDs to KEGG/ChEBI entity IDs (e.g. notebooks/test_data/HAT/annotation_df.csv)
- PiMP-exported KEGG compound pathway database (PiMP_KEGG)
- Supplementary Table S6 with T. brucei auxotrophic amino acid directions (glycine undetected)

## Outputs

- test_output.csv pathway ranking with p-value, unq_pw_F, tot_ds_F and F_coverage columns for comparisons Stage_1/Control and Stage_2/Control
- Counts of significantly changing pathways per dataset/comparison (expected 52/74 Plasma S1:C, S2:C; 42/128 and 42/143 CSF)
- Direction-of-change assignment for T. brucei auxotrophic amino acid pathways in CSF S2-vs-control (all decreasing, glycine excluded as undetected)

## How to apply

Run PALS on the HAT peak-intensity matrix (notebooks/test_data/HAT/int_df.csv) and compound-annotation matrix (annotation_df.csv) via `python pals/run.py PLAGE <int_df> <annotation_df> test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000`; PALS imputes all-zero factor values with the minimum intensity (5000), partial zeros with the factor's non-zero mean, then log-2 transforms and SciPy-standardises to zero mean/unit variance. Sort the output pathways by the p-value columns and inspect `unq_pw_F` (unique formulae per pathway), `tot_ds_F` (formula hits in dataset) and `F_coverage` (tot_ds_F/unq_pw_F). Count significantly changing pathways and compare against the reported 52/74 (Plasma S1:C, S2:C) and 42/128, 42/143 (CSF) counts. Then, in the CSF Stage_2-vs-control ranking, verify that all T. brucei auxotrophic amino acid pathway activities decrease — with glycine excluded because it was undetected in the experiment — matching Supplementary Table S6.

## Related tools

- **PALS** (Runs PLAGE decomposition and produces the pathway ranking CSV whose CSF S2-vs-control output is checked for decreasing T. brucei auxotrophic amino acid pathway activities; installed via `pip install PALS-pathway`) — github.com/glasgowcompbio/PALS
- **pandas** (Stores and passes the intensity (int_df) and annotation (annot_df) dataframes when PALS is used programmatically)
- **SciPy** (Standardises the imputed and log-2 transformed intensity matrix to zero mean and unit variance via its preprocessing module)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- test_output.csv contains the p-value, unq_pw_F, tot_ds_F and F_coverage columns
- Reproduced changing-pathway counts match the paper's 52/74 (Plasma S1:C, S2:C) and 42/128, 42/143 (CSF)
- In the CSF S2-vs-control ranking, every detected T. brucei auxotrophic amino acid pathway activity decreases (glycine excluded as undetected)
- Directions match Supplementary Table S6 amino-acid entries
- Default imputation used --min_replace 5000 with all-zero factors replaced by minimum intensity and partial zeros by the factor's non-zero mean

## Limitations

- Glycine was undetected in the experiment, so its auxotrophic behaviour cannot be verified from the data
- Peaks without compound annotations are excluded from pathway analysis, limiting pathway coverage (F_coverage = tot_ds_F/unq_pw_F)
- Multiple peak IDs may map to multiple compound IDs and vice versa due to uncertainty in peak identification
- No software version numbers or changelog are stated; use the latest development version via Git at your own risk

## Evidence

- [supplementary] all T. brucei auxotrophic amino acids detected in the experiment (i.e., all except glycine, which was undetected) were found to be decreasing in the CSF of stage 2 (S2) HAT patients: [evidence span withheld — anchor status: not_found]
- [readme] PALS command on HAT test data: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
- [readme] imputation and standardisation rules: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] output columns: "The column `unq_pw_F` lists the unique formulae found in that pathway, `tot_ds_F` lists the formula hits found in the dataset, and `F_coverage` is the proportion"
- [abstract] robustness claim: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
