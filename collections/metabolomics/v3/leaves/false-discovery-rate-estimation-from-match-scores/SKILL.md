---
name: false-discovery-rate-estimation-from-match-scores
description: Use when you have query MS/MS spectra matched to a reference library
  with a similarity score for each match and need to pick a score cutoff that bounds
  the fraction of false matches (e.g., below 10%).
when_to_use_negative:
- Input matches lack a way to classify matches as true vs. false (no reference/ground-truth
  basis for the equation-4 FDR definition), so no FDR can be computed
- The goal is to benchmark similarity algorithms against each other (e.g., entropy
  vs. 42 alternatives on NIST20) rather than to calibrate a decision threshold for
  one scorer
- Spectra are already summarized as a score-threshold/FDR table — re-running matching
  and cleaning is unnecessary
edam_operation: http://edamontology.org/operation_2238
tools:
- name: ms_entropy
  role: Provides clean_spectrum (precursor ion removal at precursor_mz - 1.6 Da, noise_threshold,
    centroiding) and calculate_entropy_similarity (ms2 tolerance 0.05 Da) used to
    score matches before FDR estimation
  repo: https://github.com/YuanyueLi/MSEntropy
- name: MSEntropy
  role: Python package implementing entropy similarity scoring whose matches feed
    the FDR threshold analysis
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds spectra as 2D np.ndarray peaks arrays ([m/z, intensity], dtype float32)
    passed into ms_entropy functions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:94e6217c9de3fd5457fa766beb37096cf6061032d39ce6c09b6786b4c302fdf6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python
  - ms_entropy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# False discovery rate estimation from match scores

## Summary

Calibrates a spectral similarity-score threshold by computing false discovery rates (equation 4 of the entropy-similarity paper) across candidate cutoffs, showing that FDR < 10% is achieved at entropy similarity score 0.75 for 37,299 experimental natural product MS/MS spectra.

## When to use

Use when you have query MS/MS spectra matched to a reference library with a similarity score for each match and need to pick a score cutoff that bounds the fraction of false matches (e.g., below 10%).

## When NOT to use

- Input matches lack a way to classify matches as true vs. false (no reference/ground-truth basis for the equation-4 FDR definition), so no FDR can be computed
- The goal is to benchmark similarity algorithms against each other (e.g., entropy vs. 42 alternatives on NIST20) rather than to calibrate a decision threshold for one scorer
- Spectra are already summarized as a score-threshold/FDR table — re-running matching and cleaning is unnecessary

## Inputs

- Query MS/MS spectra of natural products as dictionaries with 'precursor_mz' (float) and 'peaks' (2D np.ndarray of [m/z, intensity])
- Cleaned reference spectral library spectra in the same dictionary format (e.g., from MassBank of North America or GNPS)
- Set of candidate entropy similarity score thresholds to evaluate

## Outputs

- fdr_threshold_analysis.csv — table of entropy similarity score thresholds versus corresponding FDR values
- Identified cutoff (score 0.75) at which FDR falls below 10%

## How to apply

Load query natural-product spectra and library references as dictionaries with 'precursor_mz' (float) and 'peaks' (2D [m/z, intensity] array), reading files with ms_entropy.read_one_spectrum. Clean every spectrum with ms_entropy.clean_spectrum using precursor ion removal at precursor_mz - 1.6 Da, noise_threshold=0.01 (removing ions <1% of base peak), centroiding within +/-0.05 Da, and intensity normalization to sum 1 — the paper showed removing ions above precursor_mz - 1.6 considerably enhances spectral identification. Match each cleaned query against the cleaned library with ms_entropy.calculate_entropy_similarity at an ms2 tolerance of 0.05 Da to score all 37,299 matches. Then, at each candidate entropy similarity score threshold, apply the paper's FDR definition (equation 4): the fraction of matches judged false relative to total reported matches, and tabulate threshold vs. FDR into fdr_threshold_analysis.csv. Validate that the row nearest score 0.75 yields FDR < 0.10, matching the reported result; note the paper cautions that even with entropy similarity, no single threshold prevents all false-positive discoveries.

## Related tools

- **ms_entropy** (Provides clean_spectrum (precursor ion removal at precursor_mz - 1.6 Da, noise_threshold, centroiding) and calculate_entropy_similarity (ms2 tolerance 0.05 Da) used to score matches before FDR estimation) — https://github.com/YuanyueLi/MSEntropy
- **MSEntropy** (Python package implementing entropy similarity scoring whose matches feed the FDR threshold analysis) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds spectra as 2D np.ndarray peaks arrays ([m/z, intensity], dtype float32) passed into ms_entropy functions)

## Examples

```
import ms_entropy as me; sim = me.calculate_entropy_similarity(peaks_query, peaks_reference, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- The row of fdr_threshold_analysis.csv nearest entropy similarity score 0.75 shows FDR < 0.10
- All 37,299 query spectra received an entropy similarity score from the cleaned reference library at ms2 tolerance 0.05 Da
- Each cleaned spectrum had peaks above precursor_mz - 1.6 Da removed, peaks <1% of base peak removed, was centroided within +/-0.05 Da, and intensities sum to 1
- FDR is monotonically decreasing (or non-increasing) as the similarity threshold rises, consistent with equation 4's definition
- FDR values lie in [0, 1] for every tabulated threshold

## Limitations

- Even with entropy similarity scores, no single similarity threshold can be defined that would prevent false-positive discoveries — the 0.75 cutoff bounds FDR below 10% but does not eliminate false positives
- The 0.75 / <10% FDR result was established on 37,299 experimental natural product spectra; the cutoff may not transfer to other datasets or spectrum types
- No single similarity threshold prevents false positives even for entropy similarity, unlike methods with target-decoy FDR schemes

## Evidence

- [abstract] Reported FDR threshold at score 0.75: [evidence span withheld — anchor status: not_found]
