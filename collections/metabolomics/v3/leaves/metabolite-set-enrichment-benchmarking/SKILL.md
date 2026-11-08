---
name: metabolite-set-enrichment-benchmarking
description: Use when comparing pathway-ranking methods (ORA, mPLAGE, GSEA) for robustness
  to missing peak annotations, or when reproducing the synthetic decoy-feature benchmark
  from the PALS paper (DOI 10.3390/metabo11020103).
when_to_use_negative:
- The input is real experimental data already annotated and ready for a single pathway-ranking
  run — use the PALS ranking workflow directly rather than the synthetic decoy benchmark.
- The goal is transcriptomics/gene-set enrichment evaluation rather than metabolite-set
  ranking — PALS supports UniProt/ENSEMBL Reactome queries, but the synthetic decoy
  benchmark cited here was built for metabolomics peak-intensity data.
- Data are unknown metabolites grouped by fragmentation spectra (GNPS Molecular Families
  or MS2LDA Mass2Motifs) — use PALS's user-defined metabolite-set analysis, not the
  KEGG-pathway decoy simulation.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Runs pathway ranking via PLAGE/mPLAGE, ORA, and GSEA with data imputation,
    log-2 transformation, and SciPy standardisation; installed via pip install PALS-pathway
  repo: https://github.com/glasgowcompbio/PALS
- name: Python
  role: Scripting environment for generating the synthetic dataset, running 500 repetitions,
    and computing medians with pandas
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds int_df and annotation_df DataFrames passed directly to the PALS DataSource
    in programmatic use
  repo: https://github.com/glasgowcompbio/PALS
- name: SciPy
  role: Preprocessing module used to standardise the intensity matrix to zero mean
    and unit variance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:4562f3200ad596ab07a1971c05fc4a3d7b3a44ed07ba3c8fed3115bb5844ac8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-set-enrichment-benchmarking

## Summary

Benchmark metabolite-set ranking methods (mPLAGE/PLAGE, ORA, GSEA) from the PALS pipeline on a synthetic decoy-feature dataset, recording median target-pathway p-values across decoy-injection levels (0%–1000%) to quantify robustness to incomplete annotations.

## When to use

Use when comparing pathway-ranking methods (ORA, mPLAGE, GSEA) for robustness to missing peak annotations, or when reproducing the synthetic decoy-feature benchmark from the PALS paper (DOI 10.3390/metabo11020103).

## When NOT to use

- The input is real experimental data already annotated and ready for a single pathway-ranking run — use the PALS ranking workflow directly rather than the synthetic decoy benchmark.
- The goal is transcriptomics/gene-set enrichment evaluation rather than metabolite-set ranking — PALS supports UniProt/ENSEMBL Reactome queries, but the synthetic decoy benchmark cited here was built for metabolomics peak-intensity data.
- Data are unknown metabolites grouped by fragmentation spectra (GNPS Molecular Families or MS2LDA Mass2Motifs) — use PALS's user-defined metabolite-set analysis, not the KEGG-pathway decoy simulation.

## Inputs

- Synthetic peak-intensity matrix (rows = peak features with peak id in column one, columns = samples, with group line for experimental factors)
- Synthetic compound annotation matrix (two columns: peak ID and assigned KEGG/ChEBI entity_id)
- Pathway database (PiMP_KEGG export, or synthetic pathway definitions in PALS KEGG compound format)
- Experimental design dictionary specifying groups and case/control comparisons

## Outputs

- benchmark_median_pvalues.csv — table of median target-pathway p-values per method per decoy level
- Per-repetition p-value records for each of 500 repetitions per method and decoy level
- Confirmation of the mPLAGE robustness finding (lower medians at high decoy levels)

## How to apply

Generate a synthetic benchmark of seven target pathways plus 100 background pathways in PALS input format (peak-intensity CSV plus KEGG compound annotation matrix), then inject decoy features into background pathways at levels from 0% to 1000% to progressively degrade annotation completeness. Run each method (PLAGE/mPLAGE, ORA, GSEA) through the PALS pipeline with --min_replace 5000 imputation, log-2 transformation, and SciPy preprocessing standardisation to zero mean/unit variance, over 500 independent repetitions per method and decoy level. Record the target-pathway p-value per repetition and compute the median per method and decoy level. Compare against reference medians (ORA 2e-6, mPLAGE 0.0049, GSEA 0.0578 at 0% decoy; ORA 0.0083, mPLAGE 0.0003 at 100% decoy) within Monte Carlo tolerance. Confirm mPLAGE yields lower median p-values than ORA and GSEA at high decoy levels, reflecting its greater robustness to missing features, and write results to benchmark_median_pvalues.csv.

## Related tools

- **PALS** (Runs pathway ranking via PLAGE/mPLAGE, ORA, and GSEA with data imputation, log-2 transformation, and SciPy standardisation; installed via pip install PALS-pathway) — https://github.com/glasgowcompbio/PALS
- **Python** (Scripting environment for generating the synthetic dataset, running 500 repetitions, and computing medians with pandas) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds int_df and annotation_df DataFrames passed directly to the PALS DataSource in programmatic use) — https://github.com/glasgowcompbio/PALS
- **SciPy** (Preprocessing module used to standardise the intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE int_df.csv annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- At 0% decoy level, median p-values match the references within Monte Carlo tolerance: ORA 2×10−6, mPLAGE 0.0049, GSEA 0.0578.
- At 100% decoy level, ORA and mPLAGE medians match 0.0083 and 0.0003 respectively.
- mPLAGE produces lower median p-values than ORA and GSEA at high decoy levels (250%–1000%), reproducing the paper's robustness finding.
- Each median is computed from 500 independent repetitions per method and decoy level.
- Output table benchmark_median_pvalues.csv is populated for all decoy levels from 0% to 1000% and all three methods.

## Limitations

- No software version numbers are stated for PALS; results may differ across versions of the tool.
- No changelog was found, so reproducibility across repository states is not guaranteed.
- GSEA performs poorly at the 0% decoy level (median p-value 0.0578), so it should not be expected to match ORA/mPLAGE significance in this benchmark.
- Pathway coverage depends on annotation: peaks without compound annotations are excluded from pathway analysis, which is exactly the effect the decoy injection simulates.

## Evidence

- [results] At 0% decoy features, ORA and mPLAGE return median p-values of 2×10−6 and 0.0049 respectively, while GSEA performs less well with a median p-value of 0.0578.: [evidence span withheld — anchor status: not_found]
- [abstract] mPLAGE robustness finding: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] Imputation, transformation and standardisation: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] min_replace default: "--min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
- [readme] Unannotated peaks excluded: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
