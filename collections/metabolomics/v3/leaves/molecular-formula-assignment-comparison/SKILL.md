---
name: molecular-formula-assignment-comparison
description: Use when comparing top-1 molecular formula assignment counts between
  two SIRIUS versions or configurations (isotope-pattern-only vs MS1+MS/MS) on a fixed
  compound set, or when recomputing reported deltas such as +6.3 pp and wrong-formula
  counts like 792 vs 541.
when_to_use_negative:
- The task is structure identification via CSI:FingerID database search rather than
  molecular formula assignment benchmarking — this skill only compares formula-level
  top-1 results.
- Input data is not from the 3,965-compound evaluation dataset or lacks both SIRIUS
  version outputs, so the reported deltas (+26.6 pp, +6.3 pp) cannot be validated.
- Only a single SIRIUS setting (e.g. isotope pattern alone) is available, making the
  combined MS1+MS/MS comparison and wrong-formula reduction impossible to compute.
tools:
- name: SIRIUS 4
  role: Newer version whose top-1 molecular formula assignments are benchmarked against
    SIRIUS 3.0 on the 3,965-compound dataset
  repo: github.com/sirius-ms/sirius
- name: SIRIUS
  role: SIRIUS 3.0 baseline providing isotope-pattern and MS1+MS/MS formula assignment
    counts for the comparison
  repo: github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:79ad26f6a4a7b26dd7c786476cf24120308bc820325c384f20dc84286e3f4c81
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# molecular-formula-assignment-comparison

## Summary

Reproduces and verifies the SIRIUS 3.0 vs SIRIUS 4 molecular formula identification benchmark on the 3,965-compound evaluation dataset (Supplementary Fig. 6a / Supplementary Table 3), quantifying percentage-point gains in top-1 formula assignment and the drop in wrongly assigned formulas across isotope-pattern-only and MS1+MS/MS settings.

## When to use

Use when comparing top-1 molecular formula assignment counts between two SIRIUS versions or configurations (isotope-pattern-only vs MS1+MS/MS) on a fixed compound set, or when recomputing reported deltas such as +6.3 pp and wrong-formula counts like 792 vs 541.

## When NOT to use

- The task is structure identification via CSI:FingerID database search rather than molecular formula assignment benchmarking — this skill only compares formula-level top-1 results.
- Input data is not from the 3,965-compound evaluation dataset or lacks both SIRIUS version outputs, so the reported deltas (+26.6 pp, +6.3 pp) cannot be validated.
- Only a single SIRIUS setting (e.g. isotope pattern alone) is available, making the combined MS1+MS/MS comparison and wrong-formula reduction impossible to compute.

## Inputs

- Deposited source data for Supplementary Fig. 6a / Supplementary Table 3 (SIRIUS 3.0 and SIRIUS 4 top-1 formula assignment counts on 3,965 compounds)
- SIRIUS 3.0 and SIRIUS 4 formula assignment outputs (isotope-pattern-only and MS1+MS/MS settings)

## Outputs

- formula_assignment_comparison.csv containing per-setting top-1 assignment counts, percentage-point deltas (+26.6 pp isotope-only, +6.3 pp MS1+MS/MS), and wrong-assignment counts (792 vs 541, 31.7% reduction)

## How to apply

Load the deposited source data for Supplementary Fig. 6a / Supplementary Table 3 with pandas and extract the top-1 molecular formula assignment counts for both SIRIUS settings (isotope-pattern-only and combined MS1+MS/MS). Compute the percentage-point deltas between the two versions' top-1 assignment rates per setting, expecting +26.6 pp for isotope-only and +6.3 pp for MS1+MS/MS on the 3,965-compound dataset. Separately count wrongly assigned molecular formulas per version and express the reduction (792 for SIRIUS 3.0 to 541 for SIRIUS 4) as a percentage, expecting a 31.7% decrease. Assemble counts and deltas into a single comparison table and export it as formula_assignment_comparison.csv. Validate that recomputed deltas match the reported values within rounding tolerance and that the wrong-assignment counts reproduce 792 and 541 exactly.

## Related tools

- **SIRIUS 4** (Newer version whose top-1 molecular formula assignments are benchmarked against SIRIUS 3.0 on the 3,965-compound dataset) — github.com/sirius-ms/sirius
- **SIRIUS** (SIRIUS 3.0 baseline providing isotope-pattern and MS1+MS/MS formula assignment counts for the comparison) — github.com/sirius-ms/sirius

## Examples

```
import pandas as pd; df = pd.read_csv('supp_fig6a_source_data.csv'); delta_ms1msms = (df.sirius4_top1_ms1msms - df.sirius3_top1_ms1msms) / 3965 * 100; df.to_csv('formula_assignment_comparison.csv')
```

## Evaluation signals

- Recomputed isotope-pattern-only delta equals +26.6 percentage points within rounding tolerance.
- Recomputed MS1+MS/MS delta equals +6.3 percentage points within rounding tolerance.
- Wrongly assigned molecular formula counts reproduce 792 (SIRIUS 3.0) and 541 (SIRIUS 4) exactly.
- Wrong-assignment reduction computes to 31.7% (251 fewer wrong formulas out of 792).
- Comparison table covers all four setting/version combinations from Supplementary Fig. 6a on the 3,965-compound set.

## Limitations

- The isotope-pattern-only improvement (+26.6 pp) is much larger than the MS1+MS/MS improvement (+6.3 pp), so gains from SIRIUS 4 are setting-dependent and smaller when MS/MS data is already used.
- Isotope pattern analysis in SIRIUS was rewritten with stability stated only as 'hopefully more stable than before', so re-running analyses across versions may shift results.
- If MS1 data is available, SIRIUS now always uses the parent peak from MS1 for decomposition, which changes behavior relative to older configurations.

## Evidence

- [supplementary] For the combined MS1 + MS/MS setting, SIRIUS 4 improved correct molecular formula assignment by 6.3 percentage points over SIRIUS 3.0, and wrongly assigned molecular formulas fell from 792 (SIRIUS 3.0) to 541 (SIRIUS 4): [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution.: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [discussion] if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the"
