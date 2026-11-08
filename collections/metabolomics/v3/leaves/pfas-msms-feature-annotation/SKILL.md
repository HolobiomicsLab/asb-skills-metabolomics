---
name: pfas-msms-feature-annotation
description: Use when processing LC-HRMS/MS acquisitions of PFAS-containing samples
  (e.g. AFFF, serum) where you need per-feature PFAS MS/MS evidence flags and per-round
  confidence-scored (score A) annotation counts.
when_to_use_negative:
- The input is already a curated, confidence-scored feature table with per-feature
  PFAS annotations — re-running annotation would duplicate the scoring step.
- Only MS1 data without tandem MS/MS spectra are available, since class-based MS/MS
  screening and fragment screening require MS/MS evidence.
- Targeted quantification of a short list of certified PFAS standards (e.g. PFOA/PFOS-only
  assays) rather than non-targeted annotation of known, uncommon, and unknown PFAS.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3368
- http://edamontology.org/topic_0121
tools:
- name: FluoroMatch Modular
  role: Runs the per-round non-targeted PFAS pipeline (feature detection, blank filtering,
    class-based MS/MS screening, fragment screening, scoring) with FLOW <- FALSE
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch
  role: Provides the scoring/confidence framework that assigns score A (confident)
    annotations to features with PFAS MS/MS evidence
  repo: github.com/InnovativeOmics/Core-Match
- name: LipidMatch 4.2
  role: Distribution package whose LipidMatch_Libraries/Scripts directory hosts the
    Modular scripts; Lipid <- FALSE defaults to PFAS analysis
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:2b1f18f3e555f56084ea687082f45d3c6541d5788cd01a4365b3b02acc9d7215
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch
  - FluoroMatch Modular
  - LipidMatch
  - LipidMatch 4.2
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3368
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# pfas-msms-feature-annotation

## Summary

Annotates PFAS features from LC-HRMS/MS acquisitions by running FluoroMatch Modular's non-targeted pipeline (feature detection, blank filtering, class-based MS/MS and fragment screening) and assigning confidence scores, so PFAS MS/MS evidence counts and score A annotations can be tracked across injection rounds (e.g. iterative-exclusion rounds of an AFFF sample).

## When to use

Use when processing LC-HRMS/MS acquisitions of PFAS-containing samples (e.g. AFFF, serum) where you need per-feature PFAS MS/MS evidence flags and per-round confidence-scored (score A) annotation counts.

## When NOT to use

- The input is already a curated, confidence-scored feature table with per-feature PFAS annotations — re-running annotation would duplicate the scoring step.
- Only MS1 data without tandem MS/MS spectra are available, since class-based MS/MS screening and fragment screening require MS/MS evidence.
- Targeted quantification of a short list of certified PFAS standards (e.g. PFOA/PFOS-only assays) rather than non-targeted annotation of known, uncommon, and unknown PFAS.

## Inputs

- LC-HRMS/MS raw acquisitions per injection round (initial DDA injection plus iterative-exclusion rounds, e.g. four AFFF rounds)
- LipidMatch-4.2 FluoroMatch Modular distribution (FluoroMatch_Manual.docx, Trouble_Shooting_Common_Issues.docx)
- Project Scripts folder contents (e.g. genEIC.r, MS1Spectragen.r, Stats.R, Modular.r) from InnovativeOmics/Core-Match
- EPA CompTox Chemicals Dashboard PFAS structure list (8000+ structures) for exact mass matching

## Outputs

- Per-round feature list with PFAS MS/MS evidence flags (class-based MS/MS and fragment screening)
- Per-round confidence-scored annotations with score A (confident) assignments
- Round-wise counts table (e.g. iex_rounds_table.csv) with one row per injection round: features with PFAS MS/MS evidence and score A annotation counts

## How to apply

Load each LC-HRMS/MS acquisition round (e.g. the initial AFFF injection plus three iterative-exclusion rounds) into FluoroMatch Modular with FLOW <- FALSE, copying project scripts into RapidTest/LipidMatch_Libraries/Scripts/ per the Core-Match README. Run feature detection and feature blank filtering on every round, then apply class-based MS/MS screening and fragment screening to flag features carrying PFAS MS/MS evidence, supported by homologous series detection and mass defect filtering steps in the FluoroMatch 2.0 pipeline. Apply FluoroMatch's comprehensive confidence-level criterion/scoring framework to integrate evidence layers and assign score A (confident) annotations per round. Finally, count features with PFAS MS/MS evidence and score A annotations per round — the validated reference trend is 105 PFAS-evidence features in the initial AFFF injection rising to 399 after three iterative-exclusion rounds (+280% features, +139% score A) — and write these to a round-wise CSV table with one row per injection round.

## Related tools

- **FluoroMatch Modular** (Runs the per-round non-targeted PFAS pipeline (feature detection, blank filtering, class-based MS/MS screening, fragment screening, scoring) with FLOW <- FALSE) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch** (Provides the scoring/confidence framework that assigns score A (confident) annotations to features with PFAS MS/MS evidence) — github.com/InnovativeOmics/Core-Match
- **LipidMatch 4.2** (Distribution package whose LipidMatch_Libraries/Scripts directory hosts the Modular scripts; Lipid <- FALSE defaults to PFAS analysis) — github.com/InnovativeOmics/Core-Match

## Examples

```
cp Scripts/*.r RapidTest/LipidMatch_Libraries/Scripts/ && Rscript Modular.r  # with FLOW <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE
```

## Evaluation signals

- Round-wise counts table contains exactly four round rows (initial injection + three IE rounds) with per-round integer counts (row_count_equals).
- Feature counts reproduce the reported trend: 105 features with PFAS MS/MS evidence in the initial AFFF injection rising to 399 after three iterative-exclusion rounds (+280% features, output_matches_reference).
- Score A (confident) annotation counts increase by 139% relative to the initial round, with diminishing gains across subsequent IE rounds (high-abundance compounds fragmented by the fourth injection).
- Blank filtering and screening substantially reduce the candidate space — the paper reports 96% of features likely not PFAS were filtered out in the AFFF application.
- Each flagged feature carries layered evidence from the scoring framework (class-based MS/MS, fragment screening, mass defect/homologous series) rather than only exact mass matching.

## Limitations

- Confident annotations saturate: additional iterative-exclusion injections yield fewer new score A annotations once high-abundance compounds have been fragmented.
- Results depend on toggling distribution parameters correctly — with both Lipid and TWeen_pos FALSE the code defaults to PFAS analysis, so misconfiguration shifts the pipeline's intended application.
- Scoring is a confidence/annotation framework meant to reduce overreporting, not definitive identification; over a thousand AFFF PFAS remain 'likely' annotations including previously unreported species.

## Evidence
