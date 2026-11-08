---
name: speedup-ratio-calculation
description: Use when a paper reports a wall-clock runtime improvement between two
  tool versions on benchmark compound sets and you must recompute or verify the fold
  speedup (e.g., 332-fold on CHNOPS, 231-fold on BrClFI) from measured or reported
  timings.
when_to_use_negative:
- The comparison target is identification accuracy (e.g., identification rates over
  70%) rather than computational runtime — use accuracy-evaluation skills instead.
- No published baseline runtimes exist for the old version and re-running it is not
  feasible, so a fold-speedup ratio cannot be computed.
- The benchmark datasets differ in compound count or element alphabet between versions
  (e.g., comparing full 3,965 compounds against a subset run), which invalidates a
  direct ratio.
tools:
- name: SIRIUS 4
  role: New-version tool whose fragmenting-tree computation runtime is measured in
    command-line mode on both benchmark datasets
  repo: github.com/sirius-ms/sirius
- name: SIRIUS
  role: Software framework providing the releases page from which the benchmarked
    SIRIUS 4 binary is obtained
  repo: https://github.com/sirius-ms/sirius/releases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:6326389f0f0cdc4f238550800ea5770defb8946282205b573b3055c7e58f1721
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# speedup-ratio-calculation

## Summary

Computes fold wall-clock speedups between two software versions on standardized benchmark datasets — here, SIRIUS 4 versus SIRIUS 3.0 on tandem mass spectral fragmentation-tree computation over 1,553- and 3,965-compound benchmark sets.

## When to use

Use when a paper reports a wall-clock runtime improvement between two tool versions on benchmark compound sets and you must recompute or verify the fold speedup (e.g., 332-fold on CHNOPS, 231-fold on BrClFI) from measured or reported timings.

## When NOT to use

- The comparison target is identification accuracy (e.g., identification rates over 70%) rather than computational runtime — use accuracy-evaluation skills instead.
- No published baseline runtimes exist for the old version and re-running it is not feasible, so a fold-speedup ratio cannot be computed.
- The benchmark datasets differ in compound count or element alphabet between versions (e.g., comparing full 3,965 compounds against a subset run), which invalidates a direct ratio.

## Inputs

- 1,553-compound benchmark dataset (MS/MS-only, CHNOPS element alphabet)
- 3,965-compound benchmark dataset with BrClFI elements (MS1+MS/MS, isotope pattern analysis enabled)
- SIRIUS 3.0 baseline runtimes reported in Supplementary Results 2
- SIRIUS 4 release binary (command-line mode) from github.com/sirius-ms/sirius/releases

## Outputs

- Per-dataset wall-clock runtimes for SIRIUS 4 (e.g., 25 min 4 s on 3,850 BrClFI compounds)
- Fold speedup values relative to SIRIUS 3.0 baselines (target: 332-fold CHNOPS, 231-fold BrClFI)
- running_time_table.csv with per-dataset runtime and speedup values

## How to apply

Obtain the new-version binary from the official releases page (github.com/sirius-ms/sirius/releases) rather than rebuilding, so the measured binary matches the published one. Run SIRIUS 4 in command-line mode twice: once on the 1,553-compound CHNOPS benchmark (MS/MS-only, CHNOPS element alphabet) and once on the 3,965-compound BrClFI benchmark (MS1+MS/MS with isotope pattern analysis enabled), recording per-dataset wall-clock time. Take the SIRIUS 3.0 baseline runtimes from the published Supplementary Results 2 rather than re-running the old binary, then compute fold speedup as baseline time divided by SIRIUS 4 time. For the BrClFI dataset, note the comparison restricts to the 3,850 lightest compounds (SIRIUS 4: 25 min 4 s; SIRIUS 3.0: 4 d 1 h). Assemble per-dataset runtime and speedup values into running_time_table.csv and check the computed ratios against the reported 332-fold (CHNOPS) and 231-fold (BrClFI) values.

## Related tools

- **SIRIUS 4** (New-version tool whose fragmenting-tree computation runtime is measured in command-line mode on both benchmark datasets) — github.com/sirius-ms/sirius
- **SIRIUS** (Software framework providing the releases page from which the benchmarked SIRIUS 4 binary is obtained) — https://github.com/sirius-ms/sirius/releases

## Evaluation signals

- Computed CHNOPS speedup ratio reproduces the reported 332-fold increase
- Computed BrClFI speedup ratio reproduces the reported 231-fold increase (3,850 lightest compounds only)
- SIRIUS 4 BrClFI wall-clock time matches the reported 25 min 4 s and the SIRIUS 3.0 baseline matches 4 d 1 h
- running_time_table.csv contains runtime and speedup rows for both benchmark datasets (1,553 CHNOPS and 3,965 BrClFI)
- Both benchmark runs were executed in command-line mode with the same instrumentation (MS/MS-only vs MS1+MS/MS settings) as reported

## Limitations

- The BrClFI speedup is computed over only the 3,850 lightest of the 3,965 compounds, not the full dataset.
- SIRIUS 3.0 baseline timings are taken from reported values (Supplementary Results 2), not independently re-measured, so hardware differences between runs are not controlled.
- Wall-clock runtimes are sensitive to the execution environment; reproducing the exact fold speedup requires matching the benchmark protocol and possibly not the raw seconds.

## Evidence

- [abstract] We report SIRIUS 4 ( https://bio.informatik.uni-jena.de/sirius/ ), which provides a fast computational approach for molecular structure identification.: "We report SIRIUS 4 ( https://bio.informatik.uni-jena.de/sirius/ ), which provides a fast computational approach for molecular structure identification."
- [methods] SIRIUS allows for the automated and high-throughput analysis of small-compound MS data beyond elemental composition without requiring compound structures or a mass spectral database.: "SIRIUS allows for the automated and high-throughput analysis of small-compound MS data beyond elemental composition without requiring compound structures"
- [other] Run SIRIUS 4 in command-line mode on the 1,553-compound CHNOPS benchmark dataset (MS/MS-only, CHNOPS element alphabet), recording per-dataset wall-clock time: [evidence span withheld — anchor status: not_found]
- [supplementary] On the 1,553-compound MS/MS-only dataset (CHNOPS restricted), SIRIUS 4 was 332-fold faster than SIRIUS 3.0: [evidence span withheld — anchor status: not_found]
- [readme] All (including previous) releases can be found [here](https://github.com/sirius-ms/sirius/releases).: "All (including previous) releases can be found [here](https://github.com/sirius-ms/sirius/releases)."
