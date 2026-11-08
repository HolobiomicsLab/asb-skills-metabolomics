---
name: lipid-class-grouped-linear-modeling
description: Use when you have a LipiDex 2 pipeline peak table with lipid identifications,
  retention times, and fatty acyl compositions, and need a class-aware RT QC model
  to flag likely incorrect identifications.
when_to_use_negative:
- The input peak table has not yet been through LipiDex 2 peak identification and
  spectral matching (Degreaser expects those results as input)
- Most lipid classes have fewer than 5 identified examples, so the per-class regression
  cannot be fit
- Identifications lack fatty acyl composition detail (e.g. only sum compositions),
  leaving no ECN predictors
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: LipiDex 2 Degreaser QC module
  role: Implements the per-class ECN vs. RT modeling as one of its 6 QC evidence types
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Stand-alone executable (C# .NET 6) whose pipeline peak identification and
    spectral matching outputs feed the RT model
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:9842e8954f2accb7c08896dafc13bde157651d937b15121948083d3ca93521cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - C# .NET 6
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# lipid-class-grouped-linear-modeling

## Summary

Fits per-lipid-class effective carbon number (ECN) vs. retention time linear regressions — RT ~ FA carbons + alkene unsaturations — as implemented in the LipiDex 2 Degreaser QC module, to score and filter lipid identifications whose measured RT deviates strongly from the class-predicted RT.

## When to use

Use when you have a LipiDex 2 pipeline peak table with lipid identifications, retention times, and fatty acyl compositions, and need a class-aware RT QC model to flag likely incorrect identifications.

## When NOT to use

- The input peak table has not yet been through LipiDex 2 peak identification and spectral matching (Degreaser expects those results as input)
- Most lipid classes have fewer than 5 identified examples, so the per-class regression cannot be fit
- Identifications lack fatty acyl composition detail (e.g. only sum compositions), leaving no ECN predictors

## Inputs

- LipiDex 2 Peak Finder / spectral matching lipid feature results (.csv peak table with lipid identifications, retention times, and fatty acyl compositions)
- User parameter: minimum number of examples per lipid class (default 5)

## Outputs

- Per-lipid-class ECN vs. RT regression coefficients, examples used, and fit statistics (RT_model_report.csv)
- Flagged lipid identifications with large deviation from predicted RT

## How to apply

Load the lipid feature results (peak table with identifications, retention times, and fatty acyl compositions) from the peak identification and spectral matching steps of the LipiDex 2 pipeline. Parse each identification to extract fatty acyl carbon counts and alkene unsaturations, then group features by lipid class and exclude classes with fewer than the user-set minimum of examples (default 5). For each retained class, fit the effective carbon number versus retention time model (RT ~ FA carbons + alkene unsaturations), recording per-class coefficients and fit statistics. Use the fitted model to flag or filter likely incorrect identifications with a large deviation from predicted RT. On a basic computer, computing the underlying data for all 6 Degreaser visualizations across ~3000 lipid features takes less than 10 seconds, so runtimes far above that indicate a problem.

## Related tools

- **LipiDex 2 Degreaser QC module** (Implements the per-class ECN vs. RT modeling as one of its 6 QC evidence types) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Stand-alone executable (C# .NET 6) whose pipeline peak identification and spectral matching outputs feed the RT model) — https://github.com/coongroup/LipiDex-2

## Examples

```
LipiDex2.exe  # run the stand-alone executable (requires .NET 7.0), open the Degreaser QC module, and load the LipiDex 2 peak table .csv
```

## Evaluation signals

- RT_model_report.csv is produced containing class, coefficients, examples used, fit statistics, and timing
- Lipid classes with fewer than the minimum examples (default 5) are excluded from model fitting
- Total computation for all 6 Degreaser visualizations over ~3000 lipid features completes in under 10 seconds
- Identifications flagged as incorrect show a large deviation from their class-predicted RT in the ECN vs. RT plot

## Limitations

- Lipid classes with too few examples (below the user-specific minimum, default 5) cannot be modeled
- The model only applies where ECN (FA carbons + alkene unsaturations) is predictive of RT for the class; other RT behavior is not captured
- Manual validation after automated threshold filtering is optional but recommended, since the automated model alone is a flagging/filtering heuristic

## Evidence
