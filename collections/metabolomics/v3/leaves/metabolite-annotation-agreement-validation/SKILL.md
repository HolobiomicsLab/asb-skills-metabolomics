---
name: metabolite-annotation-agreement-validation
description: Use when you have M2S (or similar RT/mz/FI-based) match tables linking
  two LC-MS feature tables and independent metabolite annotations, and you need a
  ground-truth estimate of match correctness.
when_to_use_negative:
- Input datasets have few or no metabolite annotations — the article notes this validation
  strategy is 'limited by the number of annotations in our datasets', so use orthogonal
  validation strategies (FI correlation, covariate-association agreement, etc.) instead.
- You are still running the initial matching (step 1) or unique-correspondence (step
  2) stages — annotation agreement must be scored against final match tables after
  step 3 poor-match removal.
- The datasets were acquired under dissimilar LC-MS conditions such that modeled RT,
  m/z, and FI inter-dataset shifts do not apply; most missed annotated matches are
  between features behaving very differently from modeled shifts.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Produces the match tables whose annotation agreement is validated; free, open-source
    MATLAB toolbox
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which the M2S toolbox and the annotation-agreement validation
    are executed
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:150d34cac66f3085710b8e232eeeb228c8e7952dfbe8fa2325b2357efb9438ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - Azimuth DB
  - M2S (Match 2 Sets)
  - MATLAB
  - MESA
  - Rotterdam
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolite-annotation-agreement-validation

## Summary

Validates the quality of feature matches between two untargeted LC-MS metabolomics datasets by comparing MSI confidence-level-2 metabolite annotations of matched feature pairs, reporting the fraction of annotated features that were correctly matched.

## When to use

Use when you have M2S (or similar RT/mz/FI-based) match tables linking two LC-MS feature tables and independent metabolite annotations, and you need a ground-truth estimate of match correctness.

## When NOT to use

- Input datasets have few or no metabolite annotations — the article notes this validation strategy is 'limited by the number of annotations in our datasets', so use orthogonal validation strategies (FI correlation, covariate-association agreement, etc.) instead.
- You are still running the initial matching (step 1) or unique-correspondence (step 2) stages — annotation agreement must be scored against final match tables after step 3 poor-match removal.
- The datasets were acquired under dissimilar LC-MS conditions such that modeled RT, m/z, and FI inter-dataset shifts do not apply; most missed annotated matches are between features behaving very differently from modeled shifts.

## Inputs

- Final M2S match table (after step 3 poor-match removal) for the reference dataset (Dataset 1: 4953 unique matches; Dataset 2: 2324 unique matches)
- Final M2S match table for the target dataset
- Annotated feature list for the reference dataset (Dataset 1: 604 MSI confidence-level-2 annotations; Dataset 2: 87 annotations)
- Annotated feature list for the target dataset
- Feature tables with RT, m/z, and FI (feature intensity) values for both datasets

## Outputs

- annotation_match_validation.csv with columns dataset, total_annotated, correct_matches, fraction_correct (Dataset 1: 585/604 = 96.8%; Dataset 2: 79/87 = 90.8%)
- Counts of wrong matches and annotated features falling outside initial thresholds (Dataset 1: 19; Dataset 2: 8)

## How to apply

Load the final match tables produced after poor-match removal (step 3) for the reference and target datasets — e.g. Dataset 1 (RP UPLC MS ESI+, MESA reference vs Rotterdam target) with 4953 unique matches, and Dataset 2 (RP UPLC MS ESI-) with 2324 unique matches. Load the annotated feature lists, where features (including adducts and isotopologues) were annotated at Metabolomics Standards Initiative confidence level 2 using accurate mass, isotopic patterns, and MS^E fragmentation against LIPID MAPS, METLIN, HMDB, GNPS, and MassBank. For each annotated feature in the reference dataset, retrieve its matched partner from the match table and count a match as correct only if both features carry the same annotation. Tally total annotated, correct, wrong, and annotated features falling outside initial matching thresholds (Dataset 1: 19 outside; Dataset 2: 8 outside/wrong), then compute fraction correct = correct/total. Expected reference results are 585/604 = 96.8% for Dataset 1 and 79/87 = 90.8% for Dataset 2.

## Related tools

- **M2S (Match 2 Sets)** (Produces the match tables whose annotation agreement is validated; free, open-source MATLAB toolbox) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which the M2S toolbox and the annotation-agreement validation are executed) — https://github.com/rjdossan/M2S

## Evaluation signals

- Dataset 1 annotation agreement reproduces 585/604 correct matches = 96.8%.
- Dataset 2 annotation agreement reproduces 79/87 correct matches = 90.8%.
- Counts of annotated features outside initial thresholds match the reference (Dataset 1: 19 outside; Dataset 2: 8 outside/wrong).
- Summary table contains per-dataset rows with columns dataset, total_annotated, correct_matches, fraction_correct.
- Every correct match is confirmed by identical MSI confidence-level-2 annotation labels on both members of the matched pair.

## Limitations

- Validation by annotated features is limited by the number of annotations available in the datasets (604 and 87 here).
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, m/z, and FI.
- The optional step 3 (poor-match deletion) may inadvertently delete some correct matches, so agreement is measured after an analyst-driven step.

## Evidence

- [abstract] Abstract reference results: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [methods] Annotation confidence level: "Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative.15"
- [methods] Annotation databases: "databases LIPID
MAPS,16 METLIN,17 HMDB,18 GNPS,19 and MassBank.20"
- [discussion] Limitation of annotation-based validation: "while
being limited by the number of annotations in our datasets"
- [readme] Toolbox provenance: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
