---
name: matlab-toolbox-execution
description: Use when two similar LC-MS metabolomics feature tables (samples x features
  with per-feature RT, m/z, and FI medians) must be matched, e.g. for batch combination,
  discovery validation, or multicohort integration, and the M2S MATLAB toolbox is
  the chosen engine.
when_to_use_negative:
- Input data are raw instrument files rather than XCMS peak-picked feature tables
  with per-feature RT, m/z, and FI medians — peak-picking and QC (e.g. dilution-series
  R2 threshold) must precede M2S
- Datasets are not similar LC-MS metabolomics experiments/batches (e.g. different
  chromatographic platforms), where modeled inter-dataset RT/MZ/FI shifts will not
  transfer
- The goal is combining peak lists across many datasets rather than pairwise correspondence
  between two experiments, where tools like metaXCMS may fit better
tools:
- name: M2S (Match 2 Sets)
  role: Primary MATLAB toolbox executing the three-step feature matching (threshold
    matching, unique correspondence, poor-match removal)
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Runtime environment in which the M2S toolbox is executed, providing the interactive,
    graphically capable environment for threshold selection via distance plots
- name: metabCombiner
  role: Published alternative method run on the same three dataset pairs to benchmark
    M2S match counts and quality
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:49e27831ea411783c3309e17521ce9dceb02cb96164979be30dd72e9fa0d1e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - MATLAB
  - metabCombiner
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 25
---

# matlab-toolbox-execution

## Summary

Running the free, open-source MATLAB toolbox M2S (Match 2 Sets) to find feature correspondence between two untargeted LC-MS metabolomics datasets using only retention time, m/z, and feature intensity (RT, MZ, FI), with optional side-by-side benchmarking against metabCombiner.

## When to use

Use when two similar LC-MS metabolomics feature tables (samples x features with per-feature RT, m/z, and FI medians) must be matched, e.g. for batch combination, discovery validation, or multicohort integration, and the M2S MATLAB toolbox is the chosen engine.

## When NOT to use

- Input data are raw instrument files rather than XCMS peak-picked feature tables with per-feature RT, m/z, and FI medians — peak-picking and QC (e.g. dilution-series R2 threshold) must precede M2S
- Datasets are not similar LC-MS metabolomics experiments/batches (e.g. different chromatographic platforms), where modeled inter-dataset RT/MZ/FI shifts will not transfer
- The goal is combining peak lists across many datasets rather than pairwise correspondence between two experiments, where tools like metaXCMS may fit better

## Inputs

- Reference dataset feature table (samples x features, XCMS peak-picked, QC'd, with RT, m/z, FI medians per feature)
- Target dataset feature table (samples x features, XCMS peak-picked, QC'd, with RT, m/z, FI medians per feature)
- MATLAB runtime environment
- Optionally: metabCombiner R package for side-by-side comparison on the same dataset pairs

## Outputs

- Unique feature correspondence list (matched feature pairs between reference and target)
- Modeled inter-dataset shifts and residuals for RT, MZ, and FI
- Match counts and match-quality metrics per dataset pair (agreement with annotated features, e.g. confidence level 2 MSI annotations)
- Side-by-side M2S vs metabCombiner comparison table when benchmarking (rows for pairs S4, S5, S6, both methods)

## How to apply

Load the reference and target feature tables (e.g. MESA reference, Rotterdam target, peak-picked separately with XCMS, QC'd via a dilution series at R2 < 0.7 threshold, and RT-trimmed) and characterize each feature by medians of RT, m/z, and FI across all samples. In MATLAB, run M2S's three-step workflow: Step 1 matches all variables within large initial thresholds (approximately 1 min in RT and 0.025 Da in m/z, guided by visual inspection of distance plots), Step 2 finds the unique correspondence via neighbor-based shift estimation with residuals and a weighted penalty score, and the optional Step 3 detects and deletes poor matches by tightening thresholds. Decision point: whether to invoke Step 3 rests with the analyst, since it may inadvertently delete some correct matches. For benchmarking (Examples S4-S6, Datasets 4-6), run metabCombiner on the same pairs and assemble a per-pair comparison of match counts and quality; the paper reports M2S's qualitative advantage. Judge success by validation against annotated features: in the main example 585 of 604 annotated features were correct (96.8% of uniquely matched features), and a simulated example with 4755 features per dataset achieved 99.6% correct matches.

## Related tools

- **M2S (Match 2 Sets)** (Primary MATLAB toolbox executing the three-step feature matching (threshold matching, unique correspondence, poor-match removal)) — https://github.com/rjdossan/M2S
- **MATLAB** (Runtime environment in which the M2S toolbox is executed, providing the interactive, graphically capable environment for threshold selection via distance plots)
- **metabCombiner** (Published alternative method run on the same three dataset pairs to benchmark M2S match counts and quality)

## Evaluation signals

- Uniquely matched features agree with manually annotated features at high rates (main example: 585 of 604 annotated features correct, 96.8%; second example: 79 of 87 correct, 90.8%)
- On simulated data with 4755 features per dataset, at least 99.6% of matches are correct
- Initial matching thresholds fall near the paper's values (about 1 min RT window and 0.025 Da m/z window), set by visual inspection of distance plots
- Benchmark table contains rows for all three pairs (S4, S5, S6) and both methods, with match counts and quality metrics populated, reproducing M2S's qualitative advantage
- Poor matches form clusters distinguishable in RT/MZ/FI domains (e.g. a cluster at different FI consistent with column bleeding, removed by a tighter upper FI threshold)

## Limitations

- Threshold selection is subjective, guided by visual inspection of distance plots from the initial matching
- The decision on whether to use step 3 (poor-match deletion) rests with the analyst, and this optional step may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- No software version is stated for M2S; most missed annotated matches are between features behaving very differently from the modeled inter-dataset shifts of RT, MZ, and FI

## Evidence

- [abstract] M2S is a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [abstract] M2S repo availability: "a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S"
- [readme] Toolbox purpose per README: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
- [discussion] Step 3 is an analyst decision: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
- [methods] QC threshold before matching: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [intro] Only RT, MZ, FI are required: [evidence span withheld — anchor status: not_found]
