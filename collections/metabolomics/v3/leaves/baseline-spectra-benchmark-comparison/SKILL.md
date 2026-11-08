---
name: baseline-spectra-benchmark-comparison
description: Use when a de novo or library-search structure predictor has been run
  on MS2/MSn spectra and you need a head-to-head accuracy and statistical-significance
  comparison against SIRIUS4 and MetFrag baselines.
when_to_use_negative:
- When the task is to train or fine-tune the model itself rather than evaluate a finished
  predictor against baselines.
- When spectra are outside the study's domain (e.g. negative ion mode or elements
  beyond C,H,O,N,P,S,Cl,Br,I,F), since paired statistics would not reflect the baselines'
  intended operating range.
- When only one tool is available — no pairwise Wilcoxon comparison or baseline contrast
  can be computed.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: SIRIUS4
  role: 'Baseline mass spectrometry structure-identification tool (including CSI:
    FingerID) run on the same spectra to compare correct-structure and similarity
    percentages against TeFT.'
- name: MetFrag
  role: Independent baseline candidate-ranking tool run over the identical spectra
    sets with matching candidate constraints.
- name: TeFT
  role: The model under evaluation whose per-spectrum top candidates and similarity
    scores are compared against the baselines.
  repo: https://github.com/thumingo/TeFT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:bfe4f26c5304ecd1146867debeb87887a3dbc5992e430dc99445af9dbf656932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - SIRIUS4
  - TeFT
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# baseline-spectra-benchmark-comparison

## Summary

Benchmark a new MS2/MSn structure-annotation model (e.g. TeFT) against established baselines such as SIRIUS4/CSI:FingerID and MetFrag on identical spectrum sets, computing correct-structure percentages and top-ranked similarity percentages, then assess significance with Wilcoxon signed-rank tests.

## When to use

Use when a de novo or library-search structure predictor has been run on MS2/MSn spectra and you need a head-to-head accuracy and statistical-significance comparison against SIRIUS4 and MetFrag baselines.

## When NOT to use

- When the task is to train or fine-tune the model itself rather than evaluate a finished predictor against baselines.
- When spectra are outside the study's domain (e.g. negative ion mode or elements beyond C,H,O,N,P,S,Cl,Br,I,F), since paired statistics would not reflect the baselines' intended operating range.
- When only one tool is available — no pairwise Wilcoxon comparison or baseline contrast can be computed.

## Inputs

- Per-spectrum MS2/MSn test files (e.g. 660-spectrum test set, 93 CASMI 2017 spectra)
- Candidate structure lists per spectrum with the study's element and molecular-weight restrictions
- Reference correct-structure labels per spectrum
- Top-ranked candidates and similarity scores from each tool (TeFT, SIRIUS4, MetFrag)

## Outputs

- Per-tool correct-structure percentage (e.g. TeFT 30%, SIRIUS4 27.6%, MetFrag results)
- Per-tool percentage of spectra with top-ranked similarity >0.9 and >0.8
- Wilcoxon signed-rank p-values for each pairwise baseline comparison
- comparison_metrics.csv containing percentages and p-values matching Table 1

## How to apply

Load the test spectra (e.g. the 660-spectrum test set and 93 CASMI 2017 spectra) together with their candidate structure lists from the deposited spectral files. Run SIRIUS4 (including CSI: FingerID) and MetFrag on each spectrum under the same candidate constraints the study used — molecular weight <500 Da and only the elements C, H, O, N, P, S, Cl, Br, I, F. For each tool compute the percentage of spectra whose top-ranked candidate is the correct structure and the percentage whose top candidate exceeds a similarity threshold (e.g. >0.9 and >0.8). Then apply Wilcoxon signed-rank tests on the paired per-spectrum results (e.g. TeFT vs SIRIUS4, TeFT vs MetFrag) to obtain p-values matching Table 1 statistics. Write the resulting percentages and p-values to a metrics table such as comparison_metrics.csv. The rationale is that baselines must be run on the identical spectra with identical candidate database restrictions so the paired comparison is fair; the study found TeFT reached 30% correct structures (195/660) versus SIRIUS4 at 27.6% full structures and 39% similarity >0.9.

## Related tools

- **SIRIUS4** (Baseline mass spectrometry structure-identification tool (including CSI: FingerID) run on the same spectra to compare correct-structure and similarity percentages against TeFT.)
- **MetFrag** (Independent baseline candidate-ranking tool run over the identical spectra sets with matching candidate constraints.)
- **TeFT** (The model under evaluation whose per-spectrum top candidates and similarity scores are compared against the baselines.) — https://github.com/thumingo/TeFT

## Evaluation signals

- Percentages match Table 1 values: TeFT 30% correct structures (195/660), SIRIUS4 27.6% full structures and 39% similarity >0.9 on the 660-spectrum test set.
- Wilcoxon signed-rank tests produce p-values consistent with the paired per-spectrum comparisons reported in Table 1.
- Both baselines (SIRIUS4, MetFrag) are run on exactly the same spectrum sets and the same candidate constraints (MW <500 Da, ten permitted elements) as the model under test.
- Output CSV contains per-tool percentages plus p-values and is schema-valid (columns for tool, metric, value, p-value).
- Identical spectra sets are processed for each tool so the paired test is statistically valid.

## Limitations

- The comparison only reflects the study's operating domain (positive-ion mode, low-resolution spectra, MW <500 Da, ten permitted elements); other regimes may behave differently.
- The article reports that TeFT may not outperform other deep models on high-resolution spectra, so conclusions should be scoped to the tested settings.
- No formal code repository is cited for the baseline pipeline itself; reproducibility depends on the deposited spectral files and the exact SIRIUS4/MetFrag parameter settings.

## Evidence

- [results] TeFT correctly identified 30% of structures (195/660) with all similarity >0.9 at 47% and >0.8 at 67%: [evidence span withheld — anchor status: not_found]
- [discussion] SIRIUS4 … demands a higher level of mass resolution: "compared to other mass spectrometry prediction models, for instance, SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [full_text] Reproduce SIRIUS4 and MetFrag baseline comparisons for both benchmark datasets: [evidence span withheld — anchor status: not_found]
