---
name: fragment-observation-criteria-evaluation
description: Use when a feature-lipid pair has candidate fragment matches and you
  must decide, per user-selected class rules, which required fragments are observed
  and what identification level (class-only vs fatty acyl-level) the evidence supports.
when_to_use_negative:
- Input is raw vendor MS/MS data rather than a processed binary fragment matrix —
  fragment matching and 0/1 assignment (Steps 1-4) must happen first
- The lipid class has no user-defined required-fragment rule in the fragmentation-rules
  sheet, so identification cannot be justified by observed fragments
- Only precursor m/z was matched with no MS/MS scans within the RT and m/z tolerance
  of the feature — the result would be a level 4 precursor-only call, not a fragment-based
  identification
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: R tool that evaluates the required-fragment criteria per lipid type and assigns
    identification levels 1-4 to feature-lipid matches
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: R
  role: Implementation language; LipidMatch runs as R scripts with a gWidgets/tcltk
    dialogue-box interface for setting fragment thresholds
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:d8fb92083d63736a19e2b6420114fa76542cedb71f40fb6c4fd7202a1009b426
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fragment-observation-criteria-evaluation

## Summary

Rule-based evaluation of which in silico lipid fragments were actually observed in experimental MS/MS data, then assignment of an identification level (1-4) reflecting the structural resolution conferred by the matched fragments.

## When to use

Use when a feature-lipid pair has candidate fragment matches and you must decide, per user-selected class rules, which required fragments are observed and what identification level (class-only vs fatty acyl-level) the evidence supports.

## When NOT to use

- Input is raw vendor MS/MS data rather than a processed binary fragment matrix — fragment matching and 0/1 assignment (Steps 1-4) must happen first
- The lipid class has no user-defined required-fragment rule in the fragmentation-rules sheet, so identification cannot be justified by observed fragments
- Only precursor m/z was matched with no MS/MS scans within the RT and m/z tolerance of the feature — the result would be a level 4 precursor-only call, not a fragment-based identification

## Inputs

- Binary fragment matrix (Step 4 output) marking each fragment 0/1 by intensity and scan-count thresholds
- In silico lipid library for the queried lipid types (Excel .csv fragmentation libraries)
- User-selected required-fragment criteria per lipid class (Excel fragmentation-rules sheet)
- Feature table with precursor m/z and retention time per feature

## Outputs

- Identified lipids with assigned identification level 1-4 in the output CSV/feature table
- Per-feature lipid annotations distinguishing fatty acyl-constituent-level IDs (level 1) from class-only IDs (level 3)

## How to apply

Load the binary fragment matrix from Step 4 (each fragment assigned 1 if above the user-defined minimum intensity and scans threshold, 0 otherwise) and the in silico lipid library for the queried lipid types. For each feature-lipid pair, evaluate the user-selected required-fragment criteria for the lipid class from the Excel fragmentation-rules sheet — e.g. both fatty acyl anions for formate-adduct PCs, or the m/z 184.0733 head group for protonated PCs. Identify the lipid only if all necessary observed fragments are present. Then assign an identification level 1-4 based on the structural resolution conferred: level 1 for class plus fatty acyl constituents, level 2 for data-independent analysis, level 3 for class-only, and level 4 for precursor m/z without confirmatory fragments. Write identified lipids with their levels to the output CSV, so PC(16:0_18:1) at level 1 is distinguished from PC(32:1) at level 3 when only precursor plus head group is seen.

## Related tools

- **LipidMatch** (R tool that evaluates the required-fragment criteria per lipid type and assigns identification levels 1-4 to feature-lipid matches) — https://github.com/GarrettLab-UF/LipidMatch
- **R** (Implementation language; LipidMatch runs as R scripts with a gWidgets/tcltk dialogue-box interface for setting fragment thresholds)

## Examples

```
Rscript LipidMatch_FragmentCriteria.R --fragmentMatrix step4_fragments.csv --rules fragmentation_rules.xlsx --levels 1-4 --out identified_lipids.csv
```

## Evaluation signals

- Every output identification satisfies the required-fragment rule for its lipid type (e.g. both fatty acyl anions present for formate-adduct PCs; m/z 184.0733 head group present for protonated PCs)
- Each output row carries a level value in 1-4 consistent with the fragments marked observed (1) in the binary fragment matrix
- PC(32:1)-style sum-composition calls appear at level 3 while PC(16:0_18:1)-style constituent calls appear at level 1, reflecting the structural resolution actually conferred
- No lipid is identified when any necessary fragment is 0 in the fragment matrix or was not found within the ppm m/z tolerance
- Identification levels cross-check against corroboration rates reported for the workflow (92% agreement by fatty acyl constituents in positive mode, 98% in negative mode)

## Limitations

- Identification level 4 (precursor m/z only, without confirmatory fragments) reflects the weakest structural resolution and should not be treated as a fragment-confirmed ID
- Users must develop their own fragmentation libraries in Excel for full customization, so identification quality depends on library completeness per lipid type
- For co-eluting features with multiple molecules of similar m/z, fragment evidence may be shared across isobaric lipids, complicating level-1 constituent-level assignments

## Evidence

- [methods] Assign fragments 0/1 by intensity and scan thresholds: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria"
- [methods] Identification level assignment 1-4: "they are assigned a number based on whether they are identified by class and fatty acyl constituents (1), by data-independent analysis (2), only by class (3)"
- [methods] Lipids identified only when necessary fragments are observed: "Lipids are identified if they contain the necessary observed fragments."
- [abstract] Rule-based annotation rationale: "Rule-based identification allows for correct annotation of lipids based on the fragments observed"
- [readme] User-selectable required fragments per lipid type: "For each lipid type, the user can select which fragments must be observed for identification."
