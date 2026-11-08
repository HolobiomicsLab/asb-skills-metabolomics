---
name: manually-annotated-analyte-matching
description: Use when evaluating feature-alignment accuracy against a gold standard
  of manually annotated library analytes with m/z, RT and area information across
  benchmark LC–MS metabolomics datasets.
when_to_use_negative:
- Input lacks a manually annotated gold-standard library of analytes — accuracy fractions
  cannot be computed and only relative software comparisons apply
- The raw mzML files have not yet been feature-detected and aligned (this skill evaluates
  alignment results, not raw data processing)
- The goal is aligning features itself rather than evaluating an existing alignment
  against annotations
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: G-Aligner
  role: Produces the aligned consensus features whose correctness against the manually
    annotated analytes is scored as F_ACC/A_ACC
  repo: https://github.com/CSi-Studio/G-Aligner
- name: pandas
  role: Loads the annotated feature lists and alignment outputs and computes the F_ACC/A_ACC
    fractions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:901515b2a56e7bcee945a31167fac762293051616a7662c7ccee2ea7277d95be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# manually-annotated-analyte-matching

## Summary

Matches aligned consensus features from LC–MS feature alignment (e.g., G-Aligner output on TripleTOF 6600, QE_HF and MTBLS562) against a manually annotated library of analytes (m/z, RT, area) to compute alignment accuracy as F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes).

## When to use

Use when evaluating feature-alignment accuracy against a gold standard of manually annotated library analytes with m/z, RT and area information across benchmark LC–MS metabolomics datasets.

## When NOT to use

- Input lacks a manually annotated gold-standard library of analytes — accuracy fractions cannot be computed and only relative software comparisons apply
- The raw mzML files have not yet been feature-detected and aligned (this skill evaluates alignment results, not raw data processing)
- The goal is aligning features itself rather than evaluating an existing alignment against annotations

## Inputs

- Manually annotated feature lists for three public benchmark datasets (m/z, RT, area of library analytes; Zenodo doi:10.5281/zenodo.8313034)
- Feature alignment outputs (consensus features) from G-Aligner and/or comparison software (OpenMS, MZmine2, XCMS, MetaPro) in csv format with m/z, RT and area columns

## Outputs

- Per-dataset, per-method accuracy metrics: F_ACC (fraction of correctly aligned features) and A_ACC (fraction of correctly aligned analytes)
- Aggregated comparison table (e.g. solver_comparison.csv) with columns dataset, solver, F_ACC, A_ACC, running time

## How to apply

Load the manually annotated feature lists — which contain m/z, RT and area information of library analytes for the three public benchmark datasets (TripleTOF 6600, QE_HF, MTBLS562) — together with the feature-alignment outputs (consensus features per solver, e.g. G-Aligner's Greedy, VLSNS/MSG and Gurobi runs) using pandas. Match each aligned consensus feature against the annotated library analytes and record two fractions: F_ACC, the fraction of correctly aligned features, and A_ACC, the fraction of correctly aligned analytes. Repeat per dataset and per solver/method, then aggregate rows into a table with columns dataset, solver, F_ACC, A_ACC (and running time when comparing solvers). The reference annotations and evaluation methods are distributed via Zenodo (doi:10.5281/zenodo.8313034), and the G-Aligner repo ships evaluate_*.py / .java / .Rmd scripts that implement this matching per extraction software. Validation is expert review that the computed F_ACC/A_ACC reproduce the paper's ordering (G-Aligner highest accuracy, up to 9.8% and 26.6% more accurately aligned features and analytes than OpenMS, MZmine2 and XCMS).

## Related tools

- **G-Aligner** (Produces the aligned consensus features whose correctness against the manually annotated analytes is scored as F_ACC/A_ACC) — https://github.com/CSi-Studio/G-Aligner
- **pandas** (Loads the annotated feature lists and alignment outputs and computes the F_ACC/A_ACC fractions)

## Examples

```
python data/evaluate_metapro_galigner.py  # matches G-Aligner consensus features to the manually annotated analytes and computes F_ACC/A_ACC
```

## Evaluation signals

- Every aligned consensus feature matched against the library yields a defined F_ACC value in (0,1] and every analyte a defined A_ACC value in (0,1]
- Recomputed F_ACC/A_ACC reproduce the reported ordering: G-Aligner achieves up to 9.8% and 26.6% higher accurately aligned features and analytes than OpenMS, MZmine2 and XCMS
- Per-solver ordering holds: Greedy below the global solvers, VLSNS (MSG) at Gurobi-level accuracy, with VLSNS up to 9.6x faster
- Metrics are computed consistently across all three benchmark datasets (TripleTOF 6600, QE_HF, MTBLS562) with the same annotated library
- Output table schema contains exactly the dataset, solver, F_ACC, A_ACC (and running time) columns

## Limitations

- The gold standard relies on manual annotation of feature datasets, which is labor-intensive and only available for the three published benchmark datasets
- G-Aligner's main limitation is computing speed; the multidimensional assignment problem is NP-Hard, so large datasets require the acceleration methods and may still not be painless to run
- Optimized evaluation parameters are reported only in supplementary Tables S1–S3 rather than the main text

## Evidence

- [methods] annotated library defines evaluation ground truth: "We manually annotated a feature dataset for three public benchmark datasets, which contains m/z, RT, area information of library analytes"
- [abstract] accuracy benchmark on manual annotations: "G-Aligner achieved the best performance on manually annotated feature lists and untargeted extracted features of MZmine2, OpenMS and XCMS"
- [readme] where the annotations and evaluation methods are obtained: "Targeted annotation results, evaluation results and evaluation methods can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)."
- [abstract] headroom the matching should reproduce: "G-Aligner achieved the best feature alignment performance on all the three datasets with up to 9.8% and 26.6% increase in accurately aligned features and analytes"
