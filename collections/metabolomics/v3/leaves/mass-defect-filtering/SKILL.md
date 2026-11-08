---
name: mass-defect-filtering
description: Use when a blank-filtered LC-HRMS feature table from a fluorine-rich
  sample (e.g. AFFF) must be narrowed to probable PFAS using exact mass matching,
  mass defect windows, and CF2 homologous-series evidence.
when_to_use_negative:
- Input is already a curated suspect list of known PFAS where exact mass matching
  alone suffices — mass defect filtering adds little beyond catalog matching
- Analyzing lipidomics data — set Lipid <- TRUE instead, since the PFAS mass-defect
  window does not apply to lipids (README toggles Lipid/TWeen_pos for application
  type)
- Features lack fluorine-typical mass defects (e.g. purely hydrocarbon matrices) —
  the PFAS mass-defect window will not separate classes meaningfully
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FluoroMatch Flow 2.0
  role: End-to-end workflow executing feature detection, blank filtering, exact mass
    matching, mass defect filtering, and homologous series detection as sequential
    stages
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular R scripts (Modular.r) implementing the mass-defect-filtering stage
    when run standalone with FLOW <- FALSE; defaults to PFAS analysis
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:98d580b10dda54b360d9231d0c7e1ddd5819833cd9e2389cb1d8a7d6019d336d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# mass-defect-filtering

## Summary

A FluoroMatch 2.0 pipeline stage that applies mass defect filtering alongside exact mass matching to the EPA PFAS structure list and CF2-based homologous series detection to strip features that are unlikely PFAS from LC-HRMS feature tables. In the AFFF demonstration it reduced 3740 blank-filtered features to 2290 potential and 1238 probable PFAS, a 96% overall reduction.

## When to use

Use when a blank-filtered LC-HRMS feature table from a fluorine-rich sample (e.g. AFFF) must be narrowed to probable PFAS using exact mass matching, mass defect windows, and CF2 homologous-series evidence.

## When NOT to use

- Input is already a curated suspect list of known PFAS where exact mass matching alone suffices — mass defect filtering adds little beyond catalog matching
- Analyzing lipidomics data — set Lipid <- TRUE instead, since the PFAS mass-defect window does not apply to lipids (README toggles Lipid/TWeen_pos for application type)
- Features lack fluorine-typical mass defects (e.g. purely hydrocarbon matrices) — the PFAS mass-defect window will not separate classes meaningfully

## Inputs

- Vendor LC-HRMS raw files for the AFFF sample plus three solvent blanks (from the article's data deposit)
- Blank-filtered feature table CSV (~3740 rows) from the upstream feature-blank-filtering stage
- EPA CompTox Chemicals Dashboard PFAS structure list (>8000 structures)

## Outputs

- Per-feature table annotated as 'potential PFAS' (2290) or 'probable PFAS' (1238) with exact-mass, mass-defect, and homologous-series evidence
- Stage-by-stage feature count-reduction table (31,132 → 3740 → 2290 → 1238)
- Confidence-level score distribution for all candidate features (39% scored E in the AFFF case)

## How to apply

Run this as a downstream stage in FluoroMatch Flow 2.0 (or FluoroMatch Modular with FLOW <- FALSE): after feature blank filtering retains ~3740 of ~31,132 features, exact mass match against the EPA CompTox PFAS structure list (>8000 structures), then apply mass defect filtering in the PFAS window (−0.116 to 0.268, covering 98% of the EPA list, m/z 117–1189) and detect CF2-based homologous series. Features matching only exact masses are flagged as 'potential PFAS'; those with additional mass-defect/homologous-series evidence become 'probable PFAS'. The rationale is that fluorine's characteristic negative mass defect separates PFAS from most matrix interferences, enabling discovery of PFAS absent from catalogs. Feed the surviving 1238 probable-PFAS features into downstream retention-time pattern analysis and fragment screening, then compile a stage-by-stage count-reduction table mirroring Fig. 5i–ii.

## Related tools

- **FluoroMatch Flow 2.0** (End-to-end workflow executing feature detection, blank filtering, exact mass matching, mass defect filtering, and homologous series detection as sequential stages) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular R scripts (Modular.r) implementing the mass-defect-filtering stage when run standalone with FLOW <- FALSE; defaults to PFAS analysis) — github.com/InnovativeOmics/Core-Match

## Examples

```
Rscript Modular.r  # with FLOW <- FALSE, Lipid <- FALSE (README: both FALSE defaults to PFAS analysis); scripts installed at RapidTest/LipidMatch_Libraries/Scripts/
```

## Evaluation signals

- Blank-filtered feature table contains exactly 3740 rows before this stage
- Stage counts match the published Fig. 5i–ii cascade: 31,132 total → 3740 blank-filtered → 2290 potential PFAS → 1238 probable PFAS (96% reduction)
- 39% of scored features fall in confidence level E
- Probable-PFAS assignments are supported by detected CF2-based homologous series, not exact mass match alone
- Downstream fragment screening of flagged precursors runs within the PFAS mass-defect window (−0.116 to 0.268) and m/z 117–1189

## Limitations

- Mass defect filtering is only a probabilistic flag: 'potential PFAS' calls rely on exact mass match to the EPA list, and 'probable PFAS' on homologous-series evidence — chemical plausibility of novel classes (e.g. the proposed pentafluorosulfide/sulfonic-acid terminus AFFF class) is left to expert review
- The published reduction (96%) was calibrated on a legacy 1999 AFFF holding-tank sample with three solvent blanks; other matrices may show different counts
- Coverage depends on the EPA CompTox PFAS structure list (over 8000 structures); PFAS outside that list's mass-defect range may be missed

## Evidence
