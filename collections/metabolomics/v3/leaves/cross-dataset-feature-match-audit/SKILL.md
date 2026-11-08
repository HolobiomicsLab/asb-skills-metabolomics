---
name: cross-dataset-feature-match-audit
description: Use when two LC–MS metabolomics feature tables have already been matched
  (e.g., 4953 unique matches via M2S) and you must verify correctness using the subset
  of features annotated at MSI confidence level 2.
when_to_use_negative:
- Input is a raw LC–MS file or unfiltered peak list — this audit assumes M2S matching
  has already run and produced a final match table.
- Features lack manual annotations at MSI confidence level 2 (accuracy is limited
  by the number of annotations available in the datasets).
- You need to re-run the matching itself; use the M2S matching pipeline instead of
  this post-hoc audit.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Produces the feature-correspondence match tables that this audit validates
    against manual MSI level-2 annotations
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Runtime environment in which M2S runs and where the match/annotation comparison
    and summary CSV assembly are performed
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:9d23d71b4f03e6554b332e0dc91dd64357dbb6cb0f052080e92da8fb4d00ea04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-dataset-feature-match-audit

## Summary

Audit how many and what fraction of manually annotated metabolomic features were correctly matched when two untargeted LC–MS metabolomics datasets (e.g., MESA vs Rotterdam) are joined via the M2S correspondence method.

## When to use

Use when two LC–MS metabolomics feature tables have already been matched (e.g., 4953 unique matches via M2S) and you must verify correctness using the subset of features annotated at MSI confidence level 2.

## When NOT to use

- Input is a raw LC–MS file or unfiltered peak list — this audit assumes M2S matching has already run and produced a final match table.
- Features lack manual annotations at MSI confidence level 2 (accuracy is limited by the number of annotations available in the datasets).
- You need to re-run the matching itself; use the M2S matching pipeline instead of this post-hoc audit.

## Inputs

- Final M2S match table for the pair (after step 3, e.g., 4953 unique matches for Dataset 1, 2324 for Dataset 2)
- Manually annotated feature list for the reference dataset (604 annotations in Dataset 1; 87 in Dataset 2), each at MSI confidence level 2 with annotation label
- Per-dataset identifier (e.g., 'Dataset 1', 'Dataset 2') for the summary rows

## Outputs

- Per-dataset validation counts: total_annotated, correct_matches, wrong_matches, matches_outside_thresholds
- Fraction of annotated features correctly matched (e.g., 0.968 for Dataset 1, 0.908 for Dataset 2)
- annotation_match_validation.csv with columns dataset, total_annotated, correct_matches, fraction_correct

## How to apply

Load the final match tables produced after the three-step M2S pipeline (threshold match, unique correspondence, tighten-threshold poor-match deletion) — for Dataset 1 this is 4953 unique matches and for Dataset 2, 2324. Load the independently curated annotation lists: 604 annotated features for Dataset 1 and 87 for Dataset 2, each assigned at Metabolomics Standards Initiative confidence level 2 using accurate mass, isotopic patterns, and MS^E spectra against LIPID MAPS, METLIN, HMDB, GNPS, and MassBank. For every annotated feature in the reference (MESA) dataset, look up its partner in the target (Rotterdam) match table and confirm both sides carry the same annotation; a pair counts as correct only when labels agree. Also count features that fell outside initial thresholds (19 in Dataset 1; 8 in Dataset 2) as misses or wrong matches, then compute fraction_correct = correct / total_annotated per dataset (Dataset 1: 585/604 = 96.8%; Dataset 2: 79/87 = 90.8%). Write a summary CSV with columns dataset, total_annotated, correct_matches, fraction_correct.

## Related tools

- **M2S (Match 2 Sets)** (Produces the feature-correspondence match tables that this audit validates against manual MSI level-2 annotations) — https://github.com/rjdossan/M2S
- **MATLAB** (Runtime environment in which M2S runs and where the match/annotation comparison and summary CSV assembly are performed) — https://github.com/rjdossan/M2S

## Examples

```
matchValidation(Dataset1_matches, Dataset1_annotations) % MATLAB call within the M2S toolbox comparing 4953 matched pairs against 604 MSI level-2 annotations
```

## Evaluation signals

- Dataset 1 audit must reproduce 585 correct out of 604 annotated features (fraction_correct = 0.968).
- Dataset 2 audit must reproduce 79 correct out of 87 annotated features (fraction_correct = 0.908).
- Output CSV must have exactly the columns dataset, total_annotated, correct_matches, fraction_correct.
- correct_matches + wrong_matches + outside_threshold counts must equal total_annotated per dataset (e.g., 585+?+19 = 604 for Dataset 1).
- Every annotated feature counted as 'correct' must appear in the final M2S match table with an identical annotation label on both sides.

## Limitations

- Validation accuracy is limited by the number of annotations available in the datasets (only 604 and 87 annotated features were available here).
- Features falling outside initial thresholds (19 in Dataset 1, 8 in Dataset 2) are counted as misses, which lowers the achievable fraction_correct.
- The audit is a post-hoc check and cannot detect incorrect matches among unannotated features.

## Evidence

- [abstract] 4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [methods] Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative.: "Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative."
- [methods] after QC, the MESA dataset contained 10427 features and the Rotterdam dataset contained 14097.: [evidence span withheld — anchor status: not_found]
