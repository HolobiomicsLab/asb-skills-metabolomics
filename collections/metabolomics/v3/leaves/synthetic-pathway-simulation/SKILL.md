---
name: synthetic-pathway-simulation
description: Use when benchmarking pathway-ranking methods (ORA, mPLAGE/PLAGE, GSEA)
  for robustness to missing or spurious compound annotations, e.g. simulating degraded
  peak-annotation completeness before trusting method rankings on real metabolomics
  peak data.
when_to_use_negative:
- Input is real experimental peak data with verified compound annotations — the synthetic
  simulation is for method benchmarking, not for pathway ranking of an actual study.
- The goal is a single pathway-activity ranking rather than a controlled comparison
  of method robustness under injected annotation noise.
- The pathway database is not KEGG-compound-based (e.g. Reactome via UniProt/ENSEMBL
  for proteomics or transcriptomics), since the synthetic decoy design targets KEGG
  compound annotation.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0092
tools:
- name: PALS
  role: Runs ORA, PLAGE/mPLAGE, and GSEA pathway ranking on the synthetic intensity
    and annotation matrices via run_pals.py or the Python API
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds the synthetic int_df and annotation_df dataframes passed directly to
    the PALS DataSource for programmatic runs
- name: SciPy
  role: Preprocessing module used to standardise the synthetic intensity matrix to
    zero mean and unit variance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:4ce47bb058c071e3e144caaa5b068500b25bf7c89e08da3a27bde91f6cd60563
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
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# synthetic-pathway-simulation

## Summary

Builds a synthetic metabolomics benchmark (seven significantly changing target pathways plus 100 background pathways in PALS peak-intensity/KEGG-annotation format) with injected decoy features at 0–1000% levels, to quantify how robustly ORA, mPLAGE, and GSEA recover true pathway activity as annotation completeness degrades.

## When to use

Use when benchmarking pathway-ranking methods (ORA, mPLAGE/PLAGE, GSEA) for robustness to missing or spurious compound annotations, e.g. simulating degraded peak-annotation completeness before trusting method rankings on real metabolomics peak data.

## When NOT to use

- Input is real experimental peak data with verified compound annotations — the synthetic simulation is for method benchmarking, not for pathway ranking of an actual study.
- The goal is a single pathway-activity ranking rather than a controlled comparison of method robustness under injected annotation noise.
- The pathway database is not KEGG-compound-based (e.g. Reactome via UniProt/ENSEMBL for proteomics or transcriptomics), since the synthetic decoy design targets KEGG compound annotation.

## Inputs

- Synthetic peak-intensity matrix (rows = peak features, columns = samples, with a group line assigning samples to case/control factors) in PALS CSV format
- Synthetic KEGG compound annotation matrix (peak ID to C-number mappings) in PALS two-column CSV format
- Seven synthetic target pathways plus 100 background pathways (PiMP_KEGG database format)
- Decoy-feature injection schedule (0%, 100%, 250–1000% levels)
- Case/control comparison specification, e.g. --comparisons Case/Control

## Outputs

- Per-repetition target-pathway p-values for each method (PLAGE/mPLAGE, ORA, GSEA) and decoy level
- Median p-value table per method and decoy level
- benchmark_median_pvalues.csv with full reproduced-vs-reference comparison

## How to apply

Generate a synthetic dataset of seven target pathways plus 100 background pathways formatted as a peak-intensity matrix and a KEGG compound annotation matrix in PALS input format. At each decoy level from 0% to 1000%, inject decoy features into background pathways to progressively degrade annotation completeness, mimicking real-world missing peaks. Run each ranking method through the PALS pipeline with --min_replace 5000 imputation, log-2 transformation, and SciPy standardisation (zero mean, unit variance), over 500 independent repetitions per method and decoy level. Record the target pathway's p-value in every repetition and compute the median p-value per method and decoy level. Validate medians against reference values within Monte Carlo tolerance (ORA 2e-6, mPLAGE 0.0049, GSEA 0.0578 at 0% decoy; ORA 0.0083, mPLAGE 0.0003 at 100% decoy) and confirm mPLAGE yields lower median p-values than ORA and GSEA at high decoy levels.

## Related tools

- **PALS** (Runs ORA, PLAGE/mPLAGE, and GSEA pathway ranking on the synthetic intensity and annotation matrices via run_pals.py or the Python API) — github.com/glasgowcompbio/PALS
- **pandas** (Holds the synthetic int_df and annotation_df dataframes passed directly to the PALS DataSource for programmatic runs)
- **SciPy** (Preprocessing module used to standardise the synthetic intensity matrix to zero mean and unit variance)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- At 0% decoy features, reproduced median p-values match reference values within Monte Carlo tolerance: ORA 2×10−6, mPLAGE 0.0049, GSEA 0.0578.
- At 100% decoy features, reproduced medians match references: ORA 0.0083 and mPLAGE 0.0003.
- Exactly 500 independent repetitions are recorded per method per decoy level before taking the median.
- mPLAGE median p-values are lower than both ORA and GSEA at high decoy levels (250–1000%), reproducing the paper's robustness finding.
- Output CSV contains a complete method × decoy-level median p-value table written to benchmark_median_pvalues.csv.

## Limitations

- GSEA performs poorly even with complete annotations (median p-value 0.0578 at 0% decoy), so it is a weak baseline on synthetic peak data.
- PALS applies fixed imputation behaviour (zero intensities in a whole factor replaced by min_replace, partial zeros by the factor mean), which shapes the synthetic benchmark's preprocessing and cannot be bypassed.
- Results are Monte Carlo estimates; without the 500-repetition design, medians may fall outside tolerance of the reference values.
- The article does not state software version numbers, so exact reproduction may depend on the PALS development version at the time.

## Evidence

- [abstract] A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [methods] standardised using the preprocessing module in Scipy: "standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [methods] method: Pathway ranking method to use, e.g. PLAGE, ORA or GSEA.: "method: Pathway ranking method to use, e.g. PLAGE, ORA or GSEA."
- [readme] Data imputation is performed to the intensity matrix when it is loaded: "Data imputation is performed to the intensity matrix when it is loaded: if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] --min_replace: The minimum intensity value for data imputation: "--min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
