---
name: spectral-library-threshold-optimization
description: Use when spectral-matching scores (e.g., MAVEN2 hypergeometric match
  scores) against one or more in-silico lipid libraries must be converted into a single
  calibrated cutoff per library to maximize authentic-vs-spurious annotation separation.
when_to_use_negative:
- Input is already a validated, thresholded annotation table with per-library cutoffs
  applied — recalibration would be redundant
- Spectral matching was done against experimental (not in-silico) libraries, where
  the score distribution assumptions of this collapse-and-sweep approach may not hold
- The goal is peak picking or feature detection rather than judging the authenticity
  of existing spectral-match identifications
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Performs LC-MS/MS spectral matching against the lipid fragmentation libraries
    to produce per-feature hypergeometric match scores
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (.msp) whose match-score
    distributions are thresholded by this skill
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:dfc0b6c34ca8a40ba7bf04c77340625edf3f8e1ee2680a8e6f8b40ce33589516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-library-threshold-optimization

## Summary

Derives library-specific Retention Balance Point (RBP) hypergeometric score thresholds that separate authentic from spurious MS/MS spectral-match annotations, enabling aligned lipid annotation comparison between the CalicoLipids and MS-DIAL in-silico libraries searched with MAVEN2.

## When to use

Use when spectral-matching scores (e.g., MAVEN2 hypergeometric match scores) against one or more in-silico lipid libraries must be converted into a single calibrated cutoff per library to maximize authentic-vs-spurious annotation separation.

## When NOT to use

- Input is already a validated, thresholded annotation table with per-library cutoffs applied — recalibration would be redundant
- Spectral matching was done against experimental (not in-silico) libraries, where the score distribution assumptions of this collapse-and-sweep approach may not hold
- The goal is peak picking or feature detection rather than judging the authenticity of existing spectral-match identifications

## Inputs

- Per-feature MAVEN2 spectral-match annotation tables (hypergeometric scores) for standards spiked into water background
- Per-feature MAVEN2 spectral-match annotation tables for standards spiked into yeast background
- CalicoLipids in-silico lipid fragmentation library (.msp, generated via generateDB.py)
- MS-DIAL in-silico lipid library for comparison

## Outputs

- Library-specific RBP score thresholds (expected: 22.8 for CalicoLipids, 13.9 for MS-DIAL) written to rbp_thresholds.csv
- Collapsed authentic-vs-spurious separation metrics per library
- Agreement/disagreement score histograms per library

## How to apply

Load the per-feature annotation tables produced by MAVEN2 spectral matching of standards spiked into water and yeast backgrounds against each library (CalicoLipids and MS-DIAL). Bin the hypergeometric match scores into agreement and disagreement histograms per library, then collapse each histogram pair into a single authentic-vs-spurious separation metric. Sweep candidate RBP cutoffs and select the library-specific threshold maximizing the collapsed metric; the expected results are RBP thresholds of 22.8 for CalicoLipids and 13.9 for MS-DIAL. The rationale is that different in-silico libraries yield differently distributed match scores, so a shared cutoff would misclassify identifications; per-library calibration keeps cross-library annotation comparisons aligned.

## Related tools

- **MAVEN2** (Performs LC-MS/MS spectral matching against the lipid fragmentation libraries to produce per-feature hypergeometric match scores) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (.msp) whose match-score distributions are thresholded by this skill) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS"  # generate the in-silico .msp library, then sweep RBP cutoffs over MAVEN2 match-score histograms
```

## Evaluation signals

- The derived CalicoLipids RBP threshold reproduces the reported value of 22.8
- The derived MS-DIAL RBP threshold reproduces the reported value of 13.9
- rbp_thresholds.csv exists and contains one collapsed separation metric and one threshold per library
- Sweeping cutoffs around the selected threshold yields lower collapsed authentic-vs-spurious separation, confirming it is a maximum
- Histograms contain both agreement and disagreement bins for each library, so the separation metric is computable

## Limitations

- Thresholds are library-specific: a CalicoLipids cutoff (22.8) must not be reused for MS-DIAL searches and vice versa
- Thresholds were derived from standards spiked into water and yeast backgrounds; transfer to other matrices is not validated by this article
- The CalicoLipidLibrary README notes enumerating all theoretical spectra over all lipid classes may be time-consuming, and only Python 2 is supported for library generation

## Evidence

- [abstract] separate authentic from spurious metabolite identifications: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] in-silico lipidomics library comparison: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] library generation command: "Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used
to generate a subset of the library for a set of classes of interest."
- [abstract] spectral matching algorithms: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
