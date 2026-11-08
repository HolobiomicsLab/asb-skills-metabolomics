---
name: f1-score-extraction-and-method-comparison
description: Use when a compound-identification run over 2D 1H,13C HSQC spectra must
  be scored and compared across methods (MCF grid, MCF-D peak-list variant, MetaboMiner
  baseline) using F1 metrics.
when_to_use_negative:
- The task is qualitative compound identification or concentration reconstruction
  rather than benchmarking detection performance with F1 scores
- Only a single method and single spectrum are of interest, so no cross-method comparison
  table is needed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: Produces mcf-results pickle and results csv outputs from which F1 scores are
    extracted
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Back-end optimizer for the minimum cost flow problem solved during each scored
    run
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Provides the Python interface to HiGHS used for the linear optimization in
    each scored run
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: MetaboMiner
  role: 'Baseline method whose F1 scores (N907: 76.2; N926: 82.9) are the comparison
    reference in the results table'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:4db85d10fdf9575cded3fa5c09310f74db17f91157b2f6741ea1471f18365b53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - MetaboMiner
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# F1-score extraction and method comparison

## Summary

Extracts F1 detection scores from mcfNMR run outputs (mcf-results pickle/CSV) for each target spectrum and method variant, and aggregates them into a results table to compare against baselines such as MetaboMiner (e.g. N907: 83.5 vs 76.2; N926: 81.6 vs 82.9).

## When to use

Use when a compound-identification run over 2D 1H,13C HSQC spectra must be scored and compared across methods (MCF grid, MCF-D peak-list variant, MetaboMiner baseline) using F1 metrics.

## When NOT to use

- The task is qualitative compound identification or concentration reconstruction rather than benchmarking detection performance with F1 scores
- Only a single method and single spectrum are of interest, so no cross-method comparison table is needed

## Inputs

- Target spectrum csv files (columns '1H', '13C', optionally 'weights') generated via spec2csv from grid spectra (e.g. with -b binning) or peak lists
- MetaboMiner plasma (common) library csv (compound names, peak coords '1H','13C', optional 'weights')
- mcfNMR toml configuration files per target spectrum
- mcf-results pickle files and results csv files produced by mcfNMR runs

## Outputs

- Aggregated results table (CSV) with columns spectrum (N926, N907), method (MCF grid, MCF-D, MetaboMiner) and F1 score
- Per-run detection scores extracted from mcf-results pickle/csv outputs

## How to apply

After running mcfNMR on each target spectrum (e.g. N926 and N907 plasma samples) with a toml config specifying `lib`, `target`, `assignment_radius` and `detection_threshold`, read the per-run output files (mcf-results with extension '.pickle' and results with extension '.csv') and extract the F1 detection score for each spectrum and method variant. Rerun the same targets with the peak-list-based MCF-D variant — especially for the pH-disturbed sample N907 — to obtain the peak-list detection scores. Aggregate F1 scores into a single CSV table listing spectrum (N926, N907), method (MCF grid, MCF-D, MetaboMiner) and F1 score, then validate against reported baselines (MetaboMiner: N907 = 76.2, N926 = 82.9) and expected MCF values (N926 = 81.6; N907 = 83.5 grid, 81.6 peak-list) within rounding tolerance.

## Related tools

- **mcfNMR** (Produces mcf-results pickle and results csv outputs from which F1 scores are extracted) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Back-end optimizer for the minimum cost flow problem solved during each scored run) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Provides the Python interface to HiGHS used for the linear optimization in each scored run) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **MetaboMiner** (Baseline method whose F1 scores (N907: 76.2; N926: 82.9) are the comparison reference in the results table)

## Examples

```
(.mcfnmr_venv)> python -m mcfnmr -c <config_file>
```

## Evaluation signals

- Reproduced MCF F1 scores match reported values within rounding tolerance: 81.6 (N926), 83.5 and 81.6 (N907 grid and peak-list respectively)
- Results table shows the expected relative comparison: 83.5 vs 76.2 on N907 (MCF better) and 81.6 vs 82.9 on N926 (MetaboMiner better)
- Each mcfNMR run produces both a '.pickle' mcf-results file and a '.csv' results file in the configured output_dir
- F1 scores fall in a plausible range (>0.5 but <1) consistent with reported plasma performance slightly above 0.5
- Table covers both target spectra (N926, N907) and all three method variants (MCF grid, MCF-D, MetaboMiner)

## Limitations

- MCF achieves only slightly above 0.5 F1 on plasma samples, leaving room for refinement
- Performance depends on choice of parameters (assignment_radius, detection_threshold) and method variant
- Sensitivity toward peak shifts (e.g. pH-disturbed sample N907) can affect the detection scores
- Reliance on choice of parameters and method variant is a documented limitation

## Evidence

- [readme] Extraction of F1 scores from run outputs: "Output directory for the result files (mcf-results with extension '.pickle', results with extension '.csv'"
- [readme] Setting the single-pass MCF variant: "isolated_fit: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned"
- [discussion] Plasma F1 performance: "They achieved F1 scores of values slightly larger than 0.5, pointing toward a potential for further refinements"
- [discussion] MCF-D advantage on pH-disturbed sample: "the improved performance of the peak-list-based method (MCF-D) on the plasma sample, especially for the pH-disturbed sample N907"
- [discussion] Parameter choice limitation: "reliance on choice of parameters and method variant"
- [results] Reported baseline numbers: [evidence span withheld — anchor status: not_found]
