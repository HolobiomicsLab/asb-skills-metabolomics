---
name: cross-dataset-feature-matching-evaluation
description: Use when you have completed feature matching between two LC-MS metabolomics
  datasets and need to quantify match correctness — e.g. via annotated feature concordance
  or a simulated dataset with known embedded correspondences.
when_to_use_negative:
- Input is raw LC-MS data that has not yet been peak-picked with XCMS and quality-controlled
  (QC dilution series R2 <0.7 threshold) — matching evaluation presupposes a completed
  match set.
- Only a single dataset is available with no second dataset or simulated pair to match
  against — the skill requires two feature tables or a known correspondence set.
- The goal is metabolite annotation itself rather than assessing the quality of feature
  correspondences — use annotation workflows (e.g. LIPID MAPS, METLIN, HMDB searches)
  instead.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Produces the matched feature pairs whose correctness is being evaluated; also
    provides the MATLAB environment for the validation scripts.
  repo: https://github.com/rjdossan/M2S
- name: metabCombiner
  role: Published alternative method used as a comparison benchmark on three other
    dataset pairs.
- name: XCMS
  role: Used upstream to peak-pick the datasets that yield the feature tables being
    matched and validated.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:e9502f51e089e5d17bd4ed76e077a5a16a3d09de25f45c9b45f6a270e2921ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - XCMS
  - metabCombiner
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-dataset-feature-matching-evaluation

## Summary

Evaluate how well M2S matched features between two untargeted LC-MS metabolomics datasets using six orthogonal validation strategies, including ground-truth comparison on simulated data with embedded correspondences and manual annotation concordance.

## When to use

Use when you have completed feature matching between two LC-MS metabolomics datasets and need to quantify match correctness — e.g. via annotated feature concordance or a simulated dataset with known embedded correspondences.

## When NOT to use

- Input is raw LC-MS data that has not yet been peak-picked with XCMS and quality-controlled (QC dilution series R2 <0.7 threshold) — matching evaluation presupposes a completed match set.
- Only a single dataset is available with no second dataset or simulated pair to match against — the skill requires two feature tables or a known correspondence set.
- The goal is metabolite annotation itself rather than assessing the quality of feature correspondences — use annotation workflows (e.g. LIPID MAPS, METLIN, HMDB searches) instead.

## Inputs

- Final set of matched feature pairs (feature IDs, RT, m/z, optionally FI) from the M2S MATLAB toolbox output
- Ground-truth correspondence pairs embedded in the simulated dataset design (for synthetic-data validation)
- Manually annotated feature list (confidence level 2 per Metabolomics Standards Initiative) for one or both datasets

## Outputs

- Match-correctness fraction (e.g. 99.6% on simulated Dataset 3; 96.8% and 90.8% on real examples)
- Counts of true/false matches against ground truth or annotated features
- Comparison benchmark vs. metabCombiner (optional)

## How to apply

After running the M2S matching workflow (Step 1 threshold matching, Step 2 unique correspondence via shift modeling, Step 3 poor-match deletion), gauge matching quality using orthogonal validation strategies described in the article. On real datasets, compare matched features against manually annotated metabolites (confidence level 2 per the Metabolomics Standards Initiative) to compute the fraction of correct matches. On synthetic data derived from a real dataset (e.g. Dataset 3 with 4755 features per dataset), retrieve the ground-truth correspondences embedded in the data design and compute the match-correctness fraction directly against them — this yielded 99.6% correct matches for M2S. When validating against annotated features, expect roughly 96.8% (585/604) and 90.8% (79/87) correct-match rates on the two real examples; most missed annotated matches correspond to features that behave very differently from the modeled inter-dataset shifts of RT, m/z, and FI. Optionally compare results with metabCombiner on other dataset pairs to benchmark the approach. Note that threshold selection is guided by visual inspection of distance plots, and the decision to use step 3 rests with the analyst.

## Related tools

- **M2S (Match 2 Sets)** (Produces the matched feature pairs whose correctness is being evaluated; also provides the MATLAB environment for the validation scripts.) — https://github.com/rjdossan/M2S
- **metabCombiner** (Published alternative method used as a comparison benchmark on three other dataset pairs.)
- **XCMS** (Used upstream to peak-pick the datasets that yield the feature tables being matched and validated.)

## Examples

```
matlab -batch "m2s_match('mesa.csv','rotterdam.csv'); m2s_evaluate('matches.csv','ground_truth.csv')"
```

## Evaluation signals

- On simulated Dataset 3 (4755 features per dataset), the match-correctness fraction must equal 99.6% (2682 correct matches out of the detected set).
- On the main real example, 585 of 604 manually annotated features must be correctly matched (96.8%).
- On the second real example, 79 of 87 annotated features must be correctly matched (90.8%).
- Matching quality must be gauged using six orthogonal validation strategies as stated in the article.
- Comparison with metabCombiner on the three other dataset pairs should show the advantages of M2S as described in the article.

## Limitations

- Validation against annotated features is limited by the number of annotations available in the datasets.
- Threshold selection is subjective, guided by visual inspection of distance plots from the initial matching.
- The optional step 3 (poor-match deletion) rests with the analyst and may inadvertently delete some correct matches.

## Evidence

- [abstract] Six orthogonal validation strategies for match correctness: "using six orthogonal validation strategies to gauge the matching quality"
- [abstract] Simulated data correctness rate: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [abstract] Real example 1 concordance: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] Real example 2 concordance: "2324 features could be uniquely matched, with 79 (90.8%) out of 87 annotated features correctly matched"
- [methods] Annotation confidence level: "Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative"
- [methods] Simulation design: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: annotation availability: "while being limited by the number of annotations in our datasets"
- [discussion] Missing signals: subjective thresholds: "guided by visual inspection of the distance plots resulting from the initial matching"
- [discussion] Step 3 analyst discretion: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
