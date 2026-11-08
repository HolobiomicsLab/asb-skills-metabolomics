---
name: wilcoxon-signed-rank-test
description: Use when two structure-prediction methods have been run on the same paired
  set of MS2 spectra (e.g., 93 CASMI 2017 spectra or a 660-spectrum test set) and
  you need a significance test on their per-spectrum similarity distributions.
when_to_use_negative:
- The two methods were applied to different, non-paired sets of spectra — the signed-rank
  test requires paired per-spectrum observations on the identical spectrum set.
- The comparison is between a single method's score distribution and a fixed reference
  value rather than two paired per-spectrum methods.
- The input is high-resolution-spectrum benchmarking where the paper itself does not
  claim a performance advantage for TeFT (quantitative comparison with SIRIUS4/MSNovelist/MassGenie
  is not provided).
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: TeFT
  role: Method under test; its per-spectrum similarity scores form one arm of each
    paired Wilcoxon comparison
  repo: https://github.com/thumingo/TeFT
- name: SIRIUS4
  role: 'Baseline method (includes CSI: FingerID) whose paired per-spectrum scores
    are compared against TeFT via Wilcoxon signed-rank test'
- name: MetFrag
  role: Baseline database/fragment-prediction method whose paired per-spectrum scores
    are compared against TeFT via Wilcoxon signed-rank test
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:e08a31eecd8c904973da7cfffbf1fb624ff702d6d099f3cb7abb0cb897dc1fa3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - SIRIUS
  - SIRIUS4
  - TeFT
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# wilcoxon-signed-rank-test

## Summary

Applies the Wilcoxon signed-rank test to paired per-spectrum performance scores (e.g., Tanimoto/Dice structure-similarity values) to test whether two mass-spectrometry structure-prediction tools, such as TeFT vs SIRIUS4 or MetFrag, differ significantly on the same benchmark spectra.

## When to use

Use when two structure-prediction methods have been run on the same paired set of MS2 spectra (e.g., 93 CASMI 2017 spectra or a 660-spectrum test set) and you need a significance test on their per-spectrum similarity distributions.

## When NOT to use

- The two methods were applied to different, non-paired sets of spectra — the signed-rank test requires paired per-spectrum observations on the identical spectrum set.
- The comparison is between a single method's score distribution and a fixed reference value rather than two paired per-spectrum methods.
- The input is high-resolution-spectrum benchmarking where the paper itself does not claim a performance advantage for TeFT (quantitative comparison with SIRIUS4/MSNovelist/MassGenie is not provided).

## Inputs

- Paired per-spectrum structure-similarity scores (Dice/Tanimoto) for TeFT and each baseline on the same 93 CASMI 2017 spectra and/or the 660-spectrum test set
- Per-spectrum correct-structure rankings from TeFT, SIRIUS4, and MetFrag computed under matching candidate constraints (MW <500 Da; ten permitted elements)

## Outputs

- Wilcoxon signed-rank p-values comparing TeFT vs SIRIUS4 and TeFT vs MetFrag similarity distributions
- comparison_metrics.csv containing correct-structure percentages, top-ranked similarity percentages, and p-values matching Table 1

## How to apply

First run both prediction methods (e.g., TeFT via thumingo/TeFT and baselines SIRIUS4 including CSI: FingerID, and MetFrag) on identical spectra under identical candidate constraints (MW <500 Da; elements C/H/O/N/P/S/Cl/Br/I/F), producing one per-spectrum score each. Score each predicted structure against its ground truth using Tanimoto/Dice similarity, yielding paired distributions. Because the same 93 CASMI 2017 spectra (and the 660-spectrum test set) are scored by both tools, the observations are paired and non-normal, so a Wilcoxon signed-rank test — not a t-test — is appropriate. Run the test twice: TeFT vs SIRIUS4 and TeFT vs MetFrag, and report p-values alongside the headline percentages (TeFT: 12% correct / 22% mostly correct on CASMI 2017; 78% similarity >0.8; 30% correct on the 660-spectrum test set). Correct application is confirmed when the reported statistics match Table 1 (Wilcoxon p = 4.4×10⁻⁶ vs SIRIUS4; p = 2.7×10⁻¹¹ vs MetFrag).

## Related tools

- **TeFT** (Method under test; its per-spectrum similarity scores form one arm of each paired Wilcoxon comparison) — https://github.com/thumingo/TeFT
- **SIRIUS4** (Baseline method (includes CSI: FingerID) whose paired per-spectrum scores are compared against TeFT via Wilcoxon signed-rank test)
- **MetFrag** (Baseline database/fragment-prediction method whose paired per-spectrum scores are compared against TeFT via Wilcoxon signed-rank test)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(teft_dice_scores, sirius4_dice_scores)
```

## Evaluation signals

- Wilcoxon p-values fall near the published values: 4.4×10⁻⁶ (TeFT vs SIRIUS4) and 2.7×10⁻¹¹ (TeFT vs MetFrag) on the 93 CASMI 2017 MS2 spectra.
- Per-spectrum score vectors are paired 1:1 across tools (length 93 for CASMI 2017; 660 for the test set) with no missing entries.
- Headline percentages reproduce before testing: TeFT 12% correct / 22% mostly correct, >0.8 similarity at 78%, >0.9 at 39% on CASMI 2017; 30% correct (195/660) on the test set.
- Baselines were run under the same constraints as TeFT (MW <500 Da; elements C, H, O, N, P, S, Cl, Br, I, F), otherwise the paired comparison is invalid.
- comparison_metrics.csv contains the expected columns: correct-structure %, top-ranked similarity %, and per-baseline p-values matching Table 1.

## Limitations

- The paired-spectrum design restricts inference to the specific benchmark sets (93 CASMI 2017 spectra; 660-spectrum test set) and constraints used; results do not extend to high-quality high-resolution spectra, where the authors state TeFT 'may not exhibit exceptional performance advantages'.
- The test requires paired, non-independent-treatment observations; re-ranking with different candidate lists or constraints breaks the pairing and invalidates the p-values.
- Only positive ion mode (H+) spectra were used to train/evaluate TeFT, so the significance comparison is limited to positive-mode data.

## Evidence

- [results] Wilcoxon p-values of 4.4×10⁻⁶ vs SIRIUS4 and 2.7×10⁻¹¹ vs MetFrag, indicating significant differences.: [evidence span withheld — anchor status: not_found]
- [full_text] Paired per-spectrum test setup on the identical 93 spectra.: [evidence span withheld — anchor status: not_found]
- [full_text] Matching candidate constraints required for a valid paired comparison.: [evidence span withheld — anchor status: not_found]
- [readme] Readme confirmation of the TeFT method being benchmarked.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from"
