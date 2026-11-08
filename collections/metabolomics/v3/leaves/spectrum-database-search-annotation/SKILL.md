---
name: spectrum-database-search-annotation
description: Use when MS/MS (MS2) spectra must be annotated by searching a candidate
  structure database with SIRIUS4 or MetFrag, producing ranked candidate lists and
  similarity scores for baseline comparison.
when_to_use_negative:
- Input spectra are high-resolution, high-accuracy data where deep-learning models
  like MassGenie or MSNovelist are more appropriate than the low-resolution-focused
  TeFT baseline comparison.
- The goal is de novo structure generation rather than database search annotation
  — TeFT's Transformer generates complete structures from MS2 spectra and should not
  be replaced by candidate ranking.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS4
  role: 'Runs CSI: FingerID database search over test and CASMI 2017 spectra with
    the same candidate constraints as TeFT to produce the SIRIUS4 baseline (27.6%
    full structures, 39% similarity >0.9).'
- name: MetFrag
  role: Independently ranks candidates for the identical spectrum sets with matching
    candidate constraints, yielding the second baseline for Wilcoxon comparison against
    TeFT.
- name: TeFT
  role: Produces the de novo predicted structure for each spectrum; its per-spectrum
    similarity scores are the treatment arm in the Wilcoxon signed-rank tests.
  repo: github.com/thumingo/TeFT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:fe2409aa5ae90137381fb8204bda45fcb4bb887cbe7328cfed5a9dbf0bad5b2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - SIRIUS4
  - TeFT
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectrum-database-search-annotation

## Summary

Baseline database-search annotation of tandem MS spectra using SIRIUS4 (with CSI: FingerID) and MetFrag to rank candidate molecular structures, against which the de novo TeFT predictions are compared and Wilcoxon-tested.

## When to use

Use when MS/MS (MS2) spectra must be annotated by searching a candidate structure database with SIRIUS4 or MetFrag, producing ranked candidate lists and similarity scores for baseline comparison.

## When NOT to use

- Input spectra are high-resolution, high-accuracy data where deep-learning models like MassGenie or MSNovelist are more appropriate than the low-resolution-focused TeFT baseline comparison.
- The goal is de novo structure generation rather than database search annotation — TeFT's Transformer generates complete structures from MS2 spectra and should not be replaced by candidate ranking.

## Inputs

- 660-spectrum test set (tandem mass spectra)
- 93 CASMI 2017 spectra
- Corresponding candidate structure lists per spectrum
- Candidate database restricted to molecular weight < 500 Da and elements C,H,O,N,P,S,Cl,Br,I,F

## Outputs

- Per-tool correct-structure percentage and top-ranked similarity percentage (>0.9, >0.8) on each spectrum set
- Wilcoxon signed-rank p-values comparing TeFT vs SIRIUS4 and TeFT vs MetFrag (Table 1 statistics)
- comparison_metrics.csv

## How to apply

Load the 660 test-set spectra and the 93 CASMI 2017 spectra together with their candidate structure lists, then run SIRIUS4 (including CSI: FingerID) and MetFrag independently over the identical spectrum sets with matched candidate constraints: molecular weight < 500 Da and only the ten elements C, H, O, N, P, S, Cl, Br, I, F permitted. For each tool compute the percentage of spectra whose top-ranked candidate is the correct structure and the percentage with top-ranked similarity > 0.9 (and > 0.8). Compare TeFT's per-spectrum results to each baseline with Wilcoxon signed-rank significance tests on the paired per-spectrum results, matching Table 1 statistics. SIRIUS4 demands higher mass resolution and higher mass accuracy than TeFT, so the baselines are expected to underperform on low-resolution miniaturized-instrument spectra.

## Related tools

- **SIRIUS4** (Runs CSI: FingerID database search over test and CASMI 2017 spectra with the same candidate constraints as TeFT to produce the SIRIUS4 baseline (27.6% full structures, 39% similarity >0.9).)
- **MetFrag** (Independently ranks candidates for the identical spectrum sets with matching candidate constraints, yielding the second baseline for Wilcoxon comparison against TeFT.)
- **TeFT** (Produces the de novo predicted structure for each spectrum; its per-spectrum similarity scores are the treatment arm in the Wilcoxon signed-rank tests.) — github.com/thumingo/TeFT

## Examples

```
sirius --input spectra.mgf --candidates candidates.tsv --elements C,H,O,N,P,S,Cl,Br,I,F --max-mw 500; metfrag --spectrum spectra.mgf --database candidates.tsv; python wilcoxon.py --teft teft_scores.csv --baseline sirius_scores.csv
```

## Evaluation signals

- SIRIUS4 must reproduce 27.6% full-structure prediction and 39% top-ranked similarity >0.9 on the 660-spectrum test set.
- TeFT must reproduce 30% correct structures (195/660) with >0.9 similarity at 47% and >0.8 similarity at 67%.
- Wilcoxon signed-rank p-values must match Table 1 statistics for TeFT vs SIRIUS4 and TeFT vs MetFrag on both spectrum sets.
- All tools must be run under identical candidate constraints (MW < 500 Da; ten permitted elements) so the comparison is paired and fair.
- Outputs must include both benchmark datasets (660 test spectra and 93 CASMI 2017 spectra) with per-spectrum paired results.

## Limitations

- SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution and mass accuracy than low-resolution miniaturized-instrument spectra provide, limiting baseline performance.
- Mass accuracy requirement applies to MSNovelist and MassGenie too, so database-search baselines may not be directly comparable on low-resolution spectra.
- TeFT was only trained on positive ion mode (H+) spectra, so baseline comparisons cannot be extended to negative-mode data without retraining.

## Evidence

- [discussion] compared to other mass spectrometry prediction models, for instance, SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution: "compared to other mass spectrometry prediction models, for instance, SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [results] On the 660-spectrum test set, TeFT correctly identified 30% of structures (195/660) with all similarity >0.9 at 47% and >0.8 at 67%, while SIRIUS4 predicted 27.6% of full structures and 39% with similarity >0.9.: [evidence span withheld — anchor status: not_found]
- [discussion] permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
