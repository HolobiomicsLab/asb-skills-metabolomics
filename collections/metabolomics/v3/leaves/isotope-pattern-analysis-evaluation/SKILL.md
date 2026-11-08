---
name: isotope-pattern-analysis-evaluation
description: Use when reproducing or extending molecular formula assignment benchmarks
  on annotated compound sets — e.g. comparing SIRIUS 3.0 vs SIRIUS 4 or isotope-pattern-only
  vs MS1+MS/MS settings on Supplementary Fig. 6a / Supplementary Table 3 data.
when_to_use_negative:
- Input spectra lack high-resolution isotope patterns — SIRIUS ranks formulas by isotope
  patterns from mass spectra of high resolution, so low-res MS1 data breaks the premise.
- Goal is structure identification (CSI:FingerID/CANOPUS output) rather than evaluating
  molecular formula assignment accuracy.
- Data are raw spectra rather than per-compound top-1 assignment counts aggregated
  over an annotated benchmark set.
tools:
- name: SIRIUS 4
  role: Candidate scoring engine whose improved isotope-pattern ranking and MS1+MS/MS
    formula assignment is being evaluated against SIRIUS 3.0
  repo: github.com/sirius-ms/sirius
- name: SIRIUS
  role: Software framework providing isotope pattern analysis and fragmentation tree
    computation used to generate per-compound formula assignments
  repo: https://github.com/sirius-ms/sirius
- name: pandas
  role: Loads the deposited source data for Supplementary Fig. 6a / Supplementary
    Table 3 and computes assignment-count deltas
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:9fb7ba5f0f605dc4ae1342374d49123d1cfbe3d17ff7e1c4cd72267f97a30813
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIRIUS
  - SIRIUS 4
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# isotope-pattern-analysis-evaluation

## Summary

Benchmark-style evaluation of SIRIUS isotope-pattern-based molecular formula assignment, comparing top-1 correct and wrongly assigned formula counts between SIRIUS versions and scoring settings (isotope-pattern-only vs MS1+MS/MS) on a 3,965-compound annotated dataset.

## When to use

Use when reproducing or extending molecular formula assignment benchmarks on annotated compound sets — e.g. comparing SIRIUS 3.0 vs SIRIUS 4 or isotope-pattern-only vs MS1+MS/MS settings on Supplementary Fig. 6a / Supplementary Table 3 data.

## When NOT to use

- Input spectra lack high-resolution isotope patterns — SIRIUS ranks formulas by isotope patterns from mass spectra of high resolution, so low-res MS1 data breaks the premise.
- Goal is structure identification (CSI:FingerID/CANOPUS output) rather than evaluating molecular formula assignment accuracy.
- Data are raw spectra rather than per-compound top-1 assignment counts aggregated over an annotated benchmark set.

## Inputs

- Deposited source data for Supplementary Fig. 6a / Supplementary Table 3 (Excel/CSV table of top-1 formula assignment counts)
- Annotated evaluation dataset of 3,965 compounds with known molecular formulas
- High-resolution MS1 isotope patterns and MS/MS peak lists per compound

## Outputs

- formula_assignment_comparison.csv (counts and percentage-point deltas per SIRIUS version and scoring setting)
- Comparison table of correctly and wrongly assigned molecular formulas (792 vs 541 wrong assignments)

## How to apply

Load the deposited source data for Supplementary Fig. 6a / Supplementary Table 3 with pandas and extract the top-1 molecular formula assignment counts for both the isotope-pattern-only and the combined MS1+MS/MS SIRIUS settings. Compute percentage-point deltas between the two settings' top-1 rates, expecting +26.6 pp for isotope-pattern-only and +6.3 pp for MS1+MS/MS when comparing SIRIUS 4 to SIRIUS 3.0. Separately compute the reduction in wrongly assigned formulas, which must go from 792 (SIRIUS 3.0) to 541 (SIRIUS 4), a 31.7% decrease. Assemble counts and deltas into a single comparison table and write it to formula_assignment_comparison.csv. Validate that recomputed deltas match reported values within rounding tolerance and that wrong-assignment counts reproduce 792 and 541 exactly.

## Related tools

- **SIRIUS 4** (Candidate scoring engine whose improved isotope-pattern ranking and MS1+MS/MS formula assignment is being evaluated against SIRIUS 3.0) — github.com/sirius-ms/sirius
- **SIRIUS** (Software framework providing isotope pattern analysis and fragmentation tree computation used to generate per-compound formula assignments) — https://github.com/sirius-ms/sirius
- **pandas** (Loads the deposited source data for Supplementary Fig. 6a / Supplementary Table 3 and computes assignment-count deltas)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('sirius_fig6a_source_data.csv'); delta = df[df.setting=='ms1_msms'].sirius4_rate - df[df.setting=='ms1_msms'].sirius3_rate; print(delta)"
```

## Evaluation signals

- Recomputed percentage-point deltas match +26.6 pp (isotope-only) and +6.3 pp (MS1+MS/MS) within rounding tolerance.
- Wrongly assigned formula counts reproduce 792 (SIRIUS 3.0) and 541 (SIRIUS 4) exactly.
- Computed relative reduction in wrong assignments equals 31.7%.
- formula_assignment_comparison.csv contains counts and deltas for both isotope-pattern-only and MS1+MS/MS settings.
- Assignment rates are evaluated in the top-1 setting over the full 3,965-compound dataset.

## Limitations

- The isotope pattern analysis was rewritten and its authors only state it is 'hopefully more stable than before', so version-to-version deltas may partly reflect implementation changes rather than method improvements.
- Formula ranking depends on high-resolution isotope patterns; instruments or acquisition modes without usable MS1 isotope information cannot be scored in the isotope-pattern-only setting.
- Experimental support for in-source fragmentations and adducts is labeled experimental in SIRIUS, which can affect which precursor formula candidates enter the ranking.

## Evidence

- [abstract] Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets.: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [methods] SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution.: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [supplementary] For the combined MS1 + MS/MS setting, SIRIUS 4 improved correct molecular formula assignment by 6.3 percentage points over SIRIUS 3.0.: [evidence span withheld — anchor status: not_found]
- [discussion] isotope pattern analysis is now rewritten and hopefully more stable than before: "isotope pattern analysis is now rewritten and hopefully more stable than before"
