---
name: macrocycle-topology-scoring-bias-correction
description: Use when two candidate molecules (e.g., a linear molecule and its cyclic
  macro-topology variant) receive the same VInSMoC peak count score and raw scores
  alone are mis for selecting the correct topology.
when_to_use_negative:
- When the two candidate molecules have clearly different raw peak-count scores that
  already resolve the ambiguity, the P-value correction is unnecessary overhead.
- When the input spectrum is not a tandem mass spectrum that can be matched to a candidate
  molecule topology (e.g., no precursor m/z or MS/MS peak list), VInSMoC scoring and
  P-value estimation do not apply.
- When searching against chemical databases unrelated to cyclic/linear macro-topology
  variants (e.g., PubChem or COCONUT library-wide screening for novel variants), the
  P-value disambiguation is not the core analysis step.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Scores the query mass spectra against the linear and cyclic candidate molecules
    and estimates Markov chain Monte Carlo P values for each spectrum–molecule match.
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts VInSMoC; used to upload the MGF/mzXML/mzML spectrum file, select the
    chemical database, configure search/fragmentation/ion modes and tolerances, and
    submit the P-value computation task.
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:29e718fc2b78f55653b3d896b3f5db9fbabf8fd2060c6ba77d7a181ad6cf3d87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# macrocycle-topology-scoring-bias-correction

## Summary

Disambiguate spectrum–molecule matches whose raw peak-count scores are identical by recomputing VInSMoC Markov chain Monte Carlo P values for linear versus cyclic macro-topology candidates, so that topology-driven raw-score bias toward cyclic compounds is corrected.

## When to use

Use when two candidate molecules (e.g., a linear molecule and its cyclic macro-topology variant) receive the same VInSMoC peak count score and raw scores alone are misleading for selecting the correct topology.

## When NOT to use

- When the two candidate molecules have clearly different raw peak-count scores that already resolve the ambiguity, the P-value correction is unnecessary overhead.
- When the input spectrum is not a tandem mass spectrum that can be matched to a candidate molecule topology (e.g., no precursor m/z or MS/MS peak list), VInSMoC scoring and P-value estimation do not apply.
- When searching against chemical databases unrelated to cyclic/linear macro-topology variants (e.g., PubChem or COCONUT library-wide screening for novel variants), the P-value disambiguation is not the core analysis step.

## Inputs

- Mass spectrum files for the linear and cyclic macro-topology molecules (MGF, mzXML, or mzML), e.g. spectra corresponding to Supplementary Fig. 18
- Candidate molecule structures (linear and cyclic variants) available from the NPAtlas chemical database on the npanalysis.org server

## Outputs

- Peak count score for each spectrum–molecule couple (expected 18 for both topologies)
- Markov chain Monte Carlo P value for each spectrum–molecule match (linear ≈ 4.005 × 10^-19; cyclic ≈ 8.900 × 10^-14)
- Tabulated results file, e.g. vinsmoc_macrocycle_pvalues.csv

## How to apply

Run VInSMoC on the example spectra for both the linear and cyclic macro-topology molecules (Supplementary Fig. 18) using the npanalysis.org server, keeping default spectral preprocessing (peak merging within the Peak Merge Threshold with at least Minimum Peak Count peaks, and keeping the Peaks Per Window most intense peaks per Peak Filtering Window). Compute the peak count score for each spectrum–molecule couple — both topologies score 18 here. Then estimate the statistical significance of each match with VInSMoC's Markov chain Monte Carlo P value strategy. The correct linear compound should yield P ≈ 4.005 × 10^-19 while the cyclic compound yields P ≈ 8.900 × 10^-14, showing the P value does not simply follow the higher cyclic raw score. Save the tabulated scores and P values (e.g., vinsmoc_macrocycle_pvalues.csv) and confirm the P-value gap persists despite the identical raw score.

## Related tools

- **VInSMoC** (Scores the query mass spectra against the linear and cyclic candidate molecules and estimates Markov chain Monte Carlo P values for each spectrum–molecule match.) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts VInSMoC; used to upload the MGF/mzXML/mzML spectrum file, select the chemical database, configure search/fragmentation/ion modes and tolerances, and submit the P-value computation task.) — https://run.npanalysis.org

## Examples

```
Browse to https://run.npanalysis.org/#/dashboard → Projects → + → upload MGF/mzXML/mzML spectra for the linear and cyclic macro-topology molecules → Tasks → + → select VInSMoC → Input Data (spectrum file + NPAtlas database) → Task Parameters (search mode: exact, fragmentation mode: Metabolite, ion mode: positive) → Submit → open the task results to view peak count scores and MCMC P values.
```

## Evaluation signals

- Both the linear and cyclic compounds must receive an identical peak count score (18) before P values are computed.
- The linear compound's MCMC P value must be ~4.005 × 10^-19 and the cyclic compound's must be ~8.900 × 10^-14 (several orders of magnitude smaller for the linear match).
- The computed P value for the linear match must be strictly smaller than that for the cyclic match, confirming that the P value does not simply track the raw score.
- The results must be reproducible when saved as vinsmoc_macrocycle_pvalues.csv with columns for topology, peak count score, and P value.

## Limitations

- The result is demonstrated on the specific example spectra of Supplementary Fig. 18; the P-value magnitudes (4.005 × 10^-19 and 8.900 × 10^-14) are tied to that example and may not generalize to arbitrary spectra.
- VInSMoC's default peak-merging and peak-filtering preprocessing settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) are assumed; changing them alters the raw peak-count score and hence the P-value comparison.
- The raw peak-count score is the only score discussed here; VInSMoC's minimum score reporting threshold controls which hits are even displayed on the task page.

## Evidence

- [supplementary] MCMC P values differ sharply between topologies with identical raw score: [evidence span withheld — anchor status: not_found]
