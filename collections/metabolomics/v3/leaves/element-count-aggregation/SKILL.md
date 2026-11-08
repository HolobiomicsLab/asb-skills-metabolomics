---
name: element-count-aggregation
description: Use when you have a formula-annotated aligned peak table (e.g. from IDSL.UFA)
  and want to detect homologous chemical-class sub-structure patterns such as polymeric/cyclic
  chains or fixed-carbon halogenated families (PCBs, PBDEs, Cl-PFTrEAs).
when_to_use_negative:
- Input is raw MS1 spectra or an unannotated peak list — formulas must already be
  assigned (e.g. by IDSL.UFA) before element-count grouping.
- Analysis goal is per-peak formula ranking rather than family-level homolog discovery;
  use the UFA annotation step instead.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: Provides the annotated formula table and its detect_formula_sets class-grouping
    algorithm that this skill re-implements or calls.
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Upstream peak-picking step that generates the individual and aligned peaklists
    consumed by IDSL.UFA before class detection.
  repo: github.com/idslme/IDSL.IPA
- name: R
  role: Execution environment for the IDSL.UFA package and its formula-class detection
    routines.
  repo: https://cran.r-project.org/package=IDSL.UFA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:21f0c22caeefd8f489cbfd7d27d36f822e9aa2957f8742c38e239ce913d6626f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# element-count-aggregation

## Summary

Parses each annotated molecular formula into element counts and groups formulas into homologous class clusters using ΔH/ΔC chain-progressions (ratio 2 or 1/2) or fixed-carbon plus fixed Σ(H+Br+Cl+F+I) rules, as implemented in IDSL.UFA's detect_formula_sets function.

## When to use

Use when you have a formula-annotated aligned peak table (e.g. from IDSL.UFA) and want to detect homologous chemical-class sub-structure patterns such as polymeric/cyclic chains or fixed-carbon halogenated families (PCBs, PBDEs, Cl-PFTrEAs).

## When NOT to use

- Input is raw MS1 spectra or an unannotated peak list — formulas must already be assigned (e.g. by IDSL.UFA) before element-count grouping.
- Analysis goal is per-peak formula ranking rather than family-level homolog discovery; use the UFA annotation step instead.

## Inputs

- Annotated aligned molecular formula table (IDSL.UFA output CSV/TXT with formula strings per peak)
- Parsed element-count columns (C, H, N, O, S, P, F, Cl, Br, I, etc.) per formula

## Outputs

- formula_class_table.csv mapping each formula to its detected class cluster
- Distinct formula-class clusters (e.g. 7,615–32,107 classes per study)

## How to apply

Load the annotated aligned molecular formula table produced by IDSL.UFA and parse each formula string into element counts (C, H, Br, Cl, F, I, N, O, etc.). Group consecutive formulas that differ by a constant ΔH/ΔC ratio of 2 (polymeric/cyclic chain progression) or 1/2 (half-unit variant), and separately group formulas that share a fixed carbon count and a constant sum of hetero/halogen atoms Σ(H+Br+Cl+F+I), which captures fixed-carbon halogenated classes such as PCBs and PBDEs. Aggregate each cluster into a distinct formula class and record cluster membership per formula; the article reports 7,615, 18,452, and 32,107 distinct formula classes detected in four public studies (ST001154, ST001683, MTBLS1684, MTBLS254). Rationale: this database-independent grouping surfaces compound families (e.g. Cl-PFTrEAs) that formula-level annotation alone would not reveal in exposomics data.

## Related tools

- **IDSL.UFA** (Provides the annotated formula table and its detect_formula_sets class-grouping algorithm that this skill re-implements or calls.) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Upstream peak-picking step that generates the individual and aligned peaklists consumed by IDSL.UFA before class detection.) — github.com/idslme/IDSL.IPA
- **R** (Execution environment for the IDSL.UFA package and its formula-class detection routines.) — https://cran.r-project.org/package=IDSL.UFA

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")  # annotated aligned table then detect_formula_sets yields formula classes
```

## Evaluation signals

- formula_class_table.csv exists, parses as a table, and contains no duplicate formula entries.
- Every input formula is assigned to exactly one class cluster or left explicitly ungrouped — no formula is silently dropped or double-counted.
- Detected chain-progressions satisfy the stated invariant: consecutive members differ by a constant ΔH/ΔC of 2 (or 1/2 for the half-unit variant).
- Fixed-carbon clusters satisfy: identical carbon count and constant Σ(H+Br+Cl+F+I) across all member formulas.
- Class-cluster counts fall in a plausible range relative to the paper's benchmark (e.g. thousands to tens of thousands of classes for a study-sized table).

## Limitations

- Requires formulas already annotated with a consistent element alphabet; IDSL.UFAx enumerates only C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P, so elements outside that set break the grouping.
- Isotopic-profile-matching formula assignment (upstream) can miss compounds not represented in the chosen IPDB chemical space, which in turn limits class detection.
- The ΔH/ΔC = 2 / 1-2 and fixed-Σ rules are heuristics for specific sub-structure patterns; formulas that do not fit these progressions remain ungrouped.

## Evidence

- [results] IDSL.UFA detects formula classes via the 'detect_formula_sets' function using constant ΔH/ΔC ratios (2 for polymeric, 1/2 for cyclic chains) and a constant carbon count with fixed Α(H+Br+Cl+F+I), as in PCBs and PBDEs.: [evidence span withheld — anchor status: not_found]
- [readme] This package can perform elemental composition calculations using the following 15 elements : C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, and P.: [evidence span withheld — anchor status: external_doc]
- [readme] Aggregating annotated molecular formulas on the aligned peak table. This is a very unique feature that is only presented by IDSL.UFA.: [evidence span withheld — anchor status: external_doc]
