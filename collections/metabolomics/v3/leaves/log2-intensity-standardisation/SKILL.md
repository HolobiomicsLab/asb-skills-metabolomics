---
name: log2-intensity-standardisation
description: Use when feeding raw peak-intensity CSV matrices into PLAGE/ORA/GSEA
  pathway-activity scoring, since mPLAGE requires imputed, log2-scaled, zero-mean/unit-variance
  intensities before SVD decomposition.
when_to_use_negative:
- Data are already log-transformed and standardised feature intensities — re-applying
  imputation with min_replace=5000 would distort real zero/low-abundance signals
- Input is a proteomics/transcriptomics matrix destined for UniProt/ENSEMBL Reactome
  matching, where the metabolomics-tuned min_replace=5000 imputation constant may
  not be appropriate
- Analysis method other than PALS (e.g. raw ORA/GSEA implementations) that specifies
  its own normalisation scheme
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PALS
  role: Implements the imputation, log2 transformation and standardisation automatically
    when the intensity matrix is loaded into a DataSource
  repo: github.com/glasgowcompbio/PALS
- name: SciPy
  role: Its preprocessing module performs the zero-mean/unit-variance standardisation
    across samples
- name: pandas
  role: Loads and passes the int_df / annotation_df dataframes that carry the intensity
    matrix into PALS
provenance:
  source_task_ids:
  - task_008
  - task_003
  - task_007
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:04bdacecc02214e827a38ad1f569d90d0e14c583c5cbaf1a3b43593d4d1c3c57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# log2-intensity-standardisation

## Summary

PALS' preprocessing step for metabolomics peak-intensity matrices: zero intensities are imputed per experimental factor (all-zero samples get min_replace=5000, partially-zero samples get the factor's non-zero mean), then values are log2-transformed and standardised with SciPy's preprocessing module to zero mean and unit variance across samples.

## When to use

Use when feeding raw peak-intensity CSV matrices into PLAGE/ORA/GSEA pathway-activity scoring, since mPLAGE requires imputed, log2-scaled, zero-mean/unit-variance intensities before SVD decomposition.

## When NOT to use

- Data are already log-transformed and standardised feature intensities — re-applying imputation with min_replace=5000 would distort real zero/low-abundance signals
- Input is a proteomics/transcriptomics matrix destined for UniProt/ENSEMBL Reactome matching, where the metabolomics-tuned min_replace=5000 imputation constant may not be appropriate
- Analysis method other than PALS (e.g. raw ORA/GSEA implementations) that specifies its own normalisation scheme

## Inputs

- Peak-intensity CSV matrix (rows = peak features, first column peak id, columns = samples; optional group line naming experimental factors)
- Annotation CSV mapping peak IDs to KEGG/ChEBI compound IDs (two columns: row_id, entity_id)

## Outputs

- Imputed, log2-transformed, zero-mean/unit-variance standardised intensity matrix ready for PLAGE/ORA/GSEA scoring

## How to apply

Load the intensity matrix (rows = peak features, columns = samples, with an optional second 'group' line indicating the experimental factor of each sample). On loading, impute zeros: if every sample in a single experimental factor has zero intensity, replace with the minimum intensity value (default min_replace=5000); if only some samples in a factor are zero, replace those with the mean of the non-zero values in that factor. Then transform the matrix to log-2 base and standardise it using the preprocessing module in SciPy so that intensities have zero mean and unit variance across samples. This standardised matrix is what PLAGE's per-set SVD consumes, and the imputation strategy is what makes mPLAGE more robust to missing peaks than ORA or GSEA.

## Related tools

- **PALS** (Implements the imputation, log2 transformation and standardisation automatically when the intensity matrix is loaded into a DataSource) — github.com/glasgowcompbio/PALS
- **SciPy** (Its preprocessing module performs the zero-mean/unit-variance standardisation across samples)
- **pandas** (Loads and passes the int_df / annotation_df dataframes that carry the intensity matrix into PALS)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- After standardisation, the intensity matrix has zero mean and unit variance across samples (verify with a column-wise mean/std check)
- All-zero samples within an experimental factor are replaced by the min_replace value (default 5000), not by the factor mean
- Partially-zero samples within a factor equal the mean of the non-zero values in that factor
- Downstream PLAGE ranking still reproduces known hits, e.g. flavonoid-related Mass2Motifs overrepresented in Rhamnus and the 23-molecule steroid-related MF significant at p<=0.05 in the American Gut data
- Output ranking table contains p-value, unq_pw_F, tot_ds_F and F_coverage columns

## Limitations

- Normalisation choices can significantly impact pathway analysis results; PALS provides a framework to investigate this rather than prescribing one universally correct scheme
- Peaks without compound annotations are excluded from pathway analysis, so imputation/standardisation alone does not rescue unannotated features
- No software version numbers are stated in the paper/README, so reproducing the exact preprocessing behaviour depends on the repository commit used

## Evidence

- [readme] if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [methods] standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance: "standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy"
- [abstract] normalisation can have a significant impact on pathway analysis results, and PALS offers a framework to further investigate this: "normalisation can have a significant impact on pathway analysis results, and PALS offers a framework to further investigate this"
