---
name: lcms-feature-correspondence-metrics-computation
description: Use when two LC–MS metabolomics feature tables have been matched (e.g.
  via M2S) and you need a quantitative validation of match quality — fraction of correctly
  matched annotated features, counts of wrong matches, and annotated features outside
  thresholds.
when_to_use_negative:
- When datasets are simulated/synthetic and ground-truth matching is known exactly
  (use the simulated-data validation strategy instead, e.g. the 99.6% correct matches
  example)
- When the goal is to perform the matching itself rather than evaluate its quality
  — use the M2S three-step matching pipeline first
- When datasets have too few annotated features for a meaningful fraction (the paper
  notes this validation is limited by the number of annotations)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Produces the unique-match tables whose quality is being quantified by annotated-feature
    comparison
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S is implemented and where the metric computation runs
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:4c7b10dfb592869b97d7e64f9a29002f875e8d08ad7af679644a8759b65331b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcms-feature-correspondence-metrics-computation

## Summary

Compute and report match-quality metrics for untargeted LC–MS feature correspondence between two datasets, quantifying how many uniquely matched features carry consistent annotations (e.g. 585/604 = 96.8% correct in Dataset 1; 79/87 = 90.8% in Dataset 2).

## When to use

Use when two LC–MS metabolomics feature tables have been matched (e.g. via M2S) and you need a quantitative validation of match quality — fraction of correctly matched annotated features, counts of wrong matches, and annotated features outside thresholds.

## When NOT to use

- When datasets are simulated/synthetic and ground-truth matching is known exactly (use the simulated-data validation strategy instead, e.g. the 99.6% correct matches example)
- When the goal is to perform the matching itself rather than evaluate its quality — use the M2S three-step matching pipeline first
- When datasets have too few annotated features for a meaningful fraction (the paper notes this validation is limited by the number of annotations)

## Inputs

- Final unique-match table from M2S (pairs of reference/target feature indices with RT, m/z, FI)
- Manually annotated feature list per dataset (MSI level 2 annotations with compound labels)
- Count of annotated features outside initial matching thresholds

## Outputs

- Validation summary table (CSV: dataset, total_annotated, correct_matches, fraction_correct)
- Per-dataset counts of correct, wrong, and outside-threshold annotated matches

## How to apply

Load the final unique-match tables (e.g. 4953 matches for Dataset 1, 2324 for Dataset 2 after M2S step 3) and the manually annotated feature lists (assigned at MSI confidence level 2 using accurate mass, isotopic patterns, and MS^E fragmentation against LIPID MAPS, METLIN, HMDB, GNPS, and MassBank). For each annotated feature in the reference (MESA) dataset, retrieve its matched partner in the target (Rotterdam) dataset and compare annotation labels; count correct, wrong, and annotated features falling outside initial thresholds (19 in Dataset 1, 8 in Dataset 2). Compute fraction correct = correct/total annotated per dataset and assemble a summary table (CSV with dataset, total_annotated, correct_matches, fraction_correct columns).

## Related tools

- **M2S (Match 2 Sets)** (Produces the unique-match tables whose quality is being quantified by annotated-feature comparison) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S is implemented and where the metric computation runs) — https://github.com/rjdossan/M2S

## Examples

```
matlab -r "load('matches_final.mat','match_table'); load('annotations_mesa.mat','annot'); correct = sum(annot(match_table{:,1}).label == annot(match_table{:,2}).label); fprintf('%d/%d = %.1f%%\n', correct, numel(annot), 100*correct/numel(annot));"
```

## Evaluation signals

- Fraction_correct for Dataset 1 should equal 585/604 ≈ 0.968 (96.8%)
- Fraction_correct for Dataset 2 should equal 79/87 ≈ 0.908 (90.8%)
- Count of annotated features outside initial thresholds should match reported values (19 for Dataset 1; 8 for Dataset 2)
- Output CSV must contain exactly the columns dataset, total_annotated, correct_matches, fraction_correct
- Total_annotated counts must equal the number of MSI level-2 annotated features (604 and 87)

## Limitations

- Validation by annotated features is limited by the number of annotations in the datasets
- Most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI
- Optional step 3 (deleting poor matches) may inadvertently delete some correct matches, biasing the fraction correct

## Evidence

- [abstract] 585 of 604 annotated correctly matched (96.8%): "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] 79 of 87 annotated correctly matched (90.8%): "2324 features could be uniquely matched, with 79 (90.8%) out of 87 annotated features correctly matched"
- [abstract] Why some annotated matches are missed: "Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI"
- [methods] Annotations are at MSI confidence level 2: "Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative.15"
- [discussion] Validation limited by number of annotations: "while being limited by the number of annotations in our datasets"
- [discussion] Optional step 3 can remove correct matches: "this final/optional step may inadvertently delete some correct matches"
