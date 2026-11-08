---
name: fallback-default-parameter-handling
description: Use when aggregating GNPS, Sirius, and ISDB-LOTUS annotations and a tool
  returns no match or only a partial (e.g. score-without-InChI) match, which would
  otherwise break tanimoto computation.
when_to_use_negative:
- When ISDB-LOTUS returns a zero match — a zero must be preserved as novelty evidence,
  not replaced by a default score.
- When a strict GNPS match exists under q(6, 0.02) — use the real returned score rather
  than any default.
- When the goal is to re-run or tune the annotation tools themselves rather than to
  fill gaps in their outputs before K-estimation.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3364
tools:
- name: MS2DECIDE
  role: Library whose K_estimation aggregation consumes the fallback-adjusted annotations;
    fallback rules are part of its scoring design
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of per-feature match scores (strict q(6, 0.02) or iterative weighted
    analog search) that trigger the GNPS-side fallbacks when missing
- name: Sirius
  role: Source of Confidence Approximate structure annotations; the 0.5 default applies
    when Sirius proposes no annotation
- name: ISDB-LOTUS
  role: Strict-search annotation source whose zero results are intentionally kept,
    contrasted with the GNPS/Sirius fallbacks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:6f0940efddc213352672e1bf1718b05309af95e33e6f109193da79589cc0c873
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Fallback default parameter handling

## Summary

Assigns expert-defined default scores when one of the MS2DECIDE annotation tools (GNPS, Sirius, ISDB-LOTUS) fails to return a match, so that the K-value aggregation is never blocked by a missing annotation.

## When to use

Use when aggregating GNPS, Sirius, and ISDB-LOTUS annotations and a tool returns no match or only a partial (e.g. score-without-InChI) match, which would otherwise break tanimoto computation.

## When NOT to use

- When ISDB-LOTUS returns a zero match — a zero must be preserved as novelty evidence, not replaced by a default score.
- When a strict GNPS match exists under q(6, 0.02) — use the real returned score rather than any default.
- When the goal is to re-run or tune the annotation tools themselves rather than to fill gaps in their outputs before K-estimation.

## Inputs

- Per-feature GNPS molecular networking annotation results (cosine scores, masses, shared-peak counts, InChI availability)
- Sirius 6 structure_identifications.tsv annotation file
- ISDB-LOTUS strict search (0.02 Da) match results
- FBMN quantitative table (.csv) and MGF file

## Outputs

- Completed per-feature indicator table with c_g, t_g,i, t_g,s filled under fallback rules (e.g. gnps_i_indicators.csv)
- Unified filtered dataframe ranked by estimated K-values exported as a .tsv file
- empty.tsv report of features with empty annotations in the iterative GNPS workflow

## How to apply

When GNPS or Sirius returns no match, the tanimoto between GNPS and Sirius cannot be calculated, so set T_gs = T_gi = 0.7 as a neutral default. When Sirius 6 (batch mode, Confidence Approximate score) fails to propose an annotation, assign a Sirius matching score of 0.5 instead of a null value. For GNPS I, apply per-feature fallbacks: a match found with score-without-InChI only yields t_g,i = t_g,s = 0.7; a feature with no match at all is set to c_g = t_g,i = t_g,s = 0. For ISDB-LOTUS, deliberately keep the zero: because a strict 0.02 Da library search was applied, a zero answer is treated as real evidence of novelty, not as missing data. Each fallback is only applied after the corresponding tool has actually been run (GNPS strict q(6, 0.02) or the iterative relaxation ladder; Sirius 6 batch; ISDB-LOTUS via spectral_lib_matcher).

## Related tools

- **MS2DECIDE** (Library whose K_estimation aggregation consumes the fallback-adjusted annotations; fallback rules are part of its scoring design) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of per-feature match scores (strict q(6, 0.02) or iterative weighted analog search) that trigger the GNPS-side fallbacks when missing)
- **Sirius** (Source of Confidence Approximate structure annotations; the 0.5 default applies when Sirius proposes no annotation)
- **ISDB-LOTUS** (Strict-search annotation source whose zero results are intentionally kept, contrasted with the GNPS/Sirius fallbacks)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Every feature row in gnps_i_indicators.csv carries a complete set of indicator values (c_g, t_g,i, t_g,s) consistent with the Table S4 discount and fallback rules.
- Features with GNPS matches but no InChI show t_g,i = t_g,s = 0.7; features with no GNPS match at all show c_g = t_g,i = t_g,s = 0.
- Sirius-missing features carry a Sirius matching score of exactly 0.5, not NaN or 0.
- GNPS–Sirius tanimoto columns (T_gs, T_gi) equal 0.7 exactly where one of the two tools had no match.
- ISDB-LOTUS zero-match rows retain score 0 and are not overwritten by any default.

## Limitations

- The 0.7 and 0.5 defaults are expert-chosen constants; they are neutral imputations, not measured scores, and may mask genuine tool disagreement.
- GNPS iterative weighted analog search runs 27 FBMN jobs and can take up to three hours; with the 0.001 library threshold some FBMN jobs fail, and the README recommends raising the threshold (e.g. to 0.1).
- Sirius 6 structure_identifications.tsv may contain redundant features (duplicated mappingFeatureId values) that break the aggregation with a ValueError before fallbacks can be applied.
- The empty-annotation report (empty.tsv) exists precisely because fallback defaults cannot recover real InChIs/SMILES for unmatched GNPS spectra.

## Evidence

- [readme] default 0.7 for missing tanimoto between GNPS and Sirius: [evidence span withheld — anchor status: external_doc]
- [readme] 0.5 default for Sirius missing annotations: [evidence span withheld — anchor status: external_doc]
- [readme] ISDB-LOTUS zero preserved as novelty signal: [evidence span withheld — anchor status: external_doc]
- [abstract] MS2DECIDE aggregates three tools: "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
