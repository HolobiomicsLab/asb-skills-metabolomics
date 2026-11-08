---
name: msms-peak-post-processing-filtering
description: Use when a predicted ESI-MS/MS peak list (CFM-ID 4.0/3.0/2.0 output at
  10, 20, 40 eV) must be trimmed to a canonical form before comparing against experimental
  Metlin QTOF spectra with peak-match metrics.
when_to_use_negative:
- The input is already a trimmed or filtered peak list — re-applying the 80%/30-peak
  rule would distort intensity distributions and invalidate matched comparisons across
  model versions
- The spectra are EI-MS/MS or from non-QTOF instruments — the article notes CFM-ID
  does not yet handle spectra collected on instruments other than QTOF, so the QTOF-tuned
  post-processing/evaluation convention does not apply
- The goal is raw spectral cleanup of experimental data (noise removal, de-isotoping)
  rather than harmonizing predicted vs experimental peak lists for benchmark scoring
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Generates the predicted ESI-MS/MS peak lists (via fragmentation graph plus
    connectivity-matrix features) that this post-processing rule trims before benchmark
    scoring
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: CFM-ID 2.0 / CFM-ID 3.0
  role: Baseline predictors whose output peak lists must receive the identical post-processing
    filter to make the Dice/Dot Product improvement comparison matched and fair
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:06291b86a393c4ef9052c01c9f8648db1ef3dd3502ba4c896b959fb4d2e84fb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 2.0 / CFM-ID 3.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Metlin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msms-peak-post-processing-filtering

## Summary

Post-processes predicted ESI-MS/MS spectra (e.g., from CFM-ID 4.0 vs 2.0/3.0) by iteratively retaining the most intense peaks until cumulative intensity reaches 80% of the total or 30 peaks are selected, so predicted peak lists are directly comparable to experimental QTOF spectra for Dice, Precision, Recall, and re-weighted Stein's Dot Product scoring.

## When to use

Use when a predicted ESI-MS/MS peak list (CFM-ID 4.0/3.0/2.0 output at 10, 20, 40 eV) must be trimmed to a canonical form before comparing against experimental Metlin QTOF spectra with peak-match metrics.

## When NOT to use

- The input is already a trimmed or filtered peak list — re-applying the 80%/30-peak rule would distort intensity distributions and invalidate matched comparisons across model versions
- The spectra are EI-MS/MS or from non-QTOF instruments — the article notes CFM-ID does not yet handle spectra collected on instruments other than QTOF, so the QTOF-tuned post-processing/evaluation convention does not apply
- The goal is raw spectral cleanup of experimental data (noise removal, de-isotoping) rather than harmonizing predicted vs experimental peak lists for benchmark scoring

## Inputs

- Predicted ESI-MS/MS peak lists (m/z, intensity) from CFM-ID 4.0 and baseline CFM-ID 2.0/3.0 at 10, 20, and 40 eV collision energies
- Experimental Metlin 2015 QTOF ESI-MS/MS spectra ([M+H]+: 4473 spectra / 1491 metabolites; [M-H]-: 2928 spectra / 976 metabolites), non-peptide, mass <= 1000 Da

## Outputs

- Filtered predicted peak lists (<= 30 peaks, >= 80% cumulative intensity) per compound, energy level, and ionization mode
- Per-energy-level and averaged metric tables (Dice, Precision, Recall, Dot Product), e.g. written to table1_metrics.csv
- Percent-improvement summary of CFM-ID 4.0 over CFM-ID 2.0/3.0 per ionization mode

## How to apply

After CFM-ID predicts ESI-MS/MS spectra for each held-out fold at 10, 20, and 40 eV, sort the predicted peaks by intensity and iteratively keep the most intense peaks until the summed intensity reaches 80% of the total or 30 peaks are selected — this prevents low-intensity noise peaks from inflating Recall and deflating Dice for learned models that emit many weak transitions. Apply the identical post-processing rule to every model version (CFM-ID 4.0 vs 2.0/3.0 baselines) and every ionization mode ([M+H]+ and [M-H]-) so the comparison is matched. Score each filtered predicted spectrum against the experimental spectrum using peak matches defined as m/z difference < 0.01 Da or 10 ppm (whichever is reached first), then compute Dice coefficient (peak presence), Precision, Recall, and re-weighted Stein's Dot Product per energy level and per ionization mode. Average metrics across all energy levels and folds before computing percent improvement of CFM-ID 4.0 over 2.0/3.0.

## Related tools

- **CFM-ID 4.0** (Generates the predicted ESI-MS/MS peak lists (via fragmentation graph plus connectivity-matrix features) that this post-processing rule trims before benchmark scoring) — https://hub.docker.com/r/wishartlab/cfmid
- **CFM-ID 2.0 / CFM-ID 3.0** (Baseline predictors whose output peak lists must receive the identical post-processing filter to make the Dice/Dot Product improvement comparison matched and fair) — https://bitbucket.org/wishartlab/cfm-id-code/

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Every filtered predicted peak list contains at most 30 peaks and the retained peaks' summed intensity is >= 80% of the pre-filter total (or the peak budget, not the intensity threshold, was the binding constraint)
- Peak matches are computed with the stated tolerance: m/z difference < 0.01 Da or 10 ppm, whichever is reached first
- Post-filter Dice improvements of CFM-ID 4.0 over 2.0/3.0 are positive on both ionization modes and directionally consistent with published Table 1 values (~31.8% Dice on [M+H]+, ~8% on [M-H]-; ~26.7% Dot Product on [M+H]+, ~20.6% on [M-H]-)
- The identical filter and match tolerance were applied to all model versions, energy levels (10/20/40 eV), and folds of the 10-fold cross-validation
- Per-metric values fall within reasonable tolerance of the published Table 1 values for the corresponding energy level and ionization mode

## Limitations

- The 80%-of-total-intensity / 30-peak cutoff was tuned for QTOF Metlin ESI-MS/MS benchmarking; other instrument types or libraries may need different truncation parameters
- The filter only addresses comparability of peak lists — the article acknowledges there is still room for improvement in both accuracy and runtime, and deeper fragmentation graphs add computational complexity
- Applying the filter cannot recover accuracy lost to poor fragmentation prediction; it only prevents unbounded peak lists from skewing Precision/Recall trade-offs

## Evidence

- [methods] Post-processing truncation rule applied to every predicted spectrum before scoring: [evidence span withheld — anchor status: not_found]
- [methods] Peak-match tolerance used to score filtered predicted spectra against experimental spectra: [evidence span withheld — anchor status: not_found]
