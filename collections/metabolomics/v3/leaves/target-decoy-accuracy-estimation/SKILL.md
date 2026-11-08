---
name: target-decoy-accuracy-estimation
description: Use when designing or validating a targeted lipidomics assay where you
  must decide how many fragment transitions per lipid (plus which polarity modes)
  suffice for unambiguous identification against a known background lipidome, and
  when reproducing a target-decoy accuracy figure from a PRM/MRM.
when_to_use_negative:
- LipidCreator provides no dedicated scoring algorithm to determine false discovery
  rates and intentionally does not address coeluting/isobaric lipids or peak area
  integration, so do not use this skill as an FDR scoring or peak-picking substitute.
- The background lipidome is unknown or the study system has no species list (e.g.
  uncharacterized bacterial/archaeal lipidomes where standards are lacking), since
  unambiguous-match probability cannot be computed without a background matrix.
- The assay involves stereoisomer resolution — the article notes no large-scale methods
  exist to cope with stereoisomers due to the lack of isotope-coded standards.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidCreator
  role: Generates the target and decoy transition lists and in silico fragment libraries
    (MS/2 fragment masses for over 60 lipid classes) used to build the background
    matrix for the probability calculation and the PRM assay.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts the LipidCreator-generated targeted PRM/MRM experiment and processes
    the resulting target-decoy identification data used for the accuracy ratio.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:a74c1b8e6480e391e553c02ac2edad64f10eab4b88cd52a57c5fd2e85b6ddc17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# target-decoy-accuracy-estimation

## Summary

Estimates how many MS/2 fragments (and polarity modes) are needed to unambiguously identify lipid species by computing target-decoy random-match probabilities over a background lipidome (yeast), and compares those probabilities against the reported LC/ESI-PRM target-decoy identification accuracy (e.g. 98% with three fragments).

## When to use

Use when designing or validating a targeted lipidomics assay where you must decide how many fragment transitions per lipid (plus which polarity modes) suffice for unambiguous identification against a known background lipidome, and when reproducing a target-decoy accuracy figure from a PRM/MRM validation run.

## When NOT to use

- LipidCreator provides no dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids or peak area integration, so do not use this skill as an FDR scoring or peak-picking substitute.
- The background lipidome is unknown or the study system has no species list (e.g. uncharacterized bacterial/archaeal lipidomes where standards are lacking), since unambiguous-match probability cannot be computed without a background matrix.
- The assay involves stereoisomer resolution — the article notes no large-scale methods exist to cope with stereoisomers due to the lack of isotope-coded standards.

## Inputs

- Yeast (or other organism) lipidome species list serving as the background matrix
- Enumerated candidate lipid species with precursor and fragment masses (LipidCreator-computed MS/2 fragment masses)
- Target-decoy LC/ESI-PRM validation results (true-target vs false-decoy identification counts, positive and negative polarity modes)

## Outputs

- validation_results.csv containing fragment-count-dependent match probabilities (at ±0.5 Da and ±2.5 ppm) and the comparison of computed probabilities to reported accuracy ratios
- Accuracy ratio estimate at three fragments across both polarity modes with a discrepancy flag if it deviates from the reported ~98% figure

## How to apply

Load the organism's lipidome species list (e.g. the yeast lipidome) as the background matrix and enumerate all candidate lipid species with their fragment masses, which LipidCreator can compute for over 60 lipid classes. For each fragment count (0, 1, 2, 3), compute the probability that a random precursor-fragment pair matches unambiguously within the background at ±0.5 Da tolerance, then repeat at ±2.5 ppm to quantify the effect of high-resolution filtering. Separately, load the published target-decoy LC/ESI-PRM validation results (true-target vs false-decoy counts) acquired, per the article's workflow, on a Q-Exactive HF in PRM mode, and compare computed probabilities against the reported accuracy ratio. The article's own finding is that 'multiple fragments and both polarity modes are required for unequivocal identification of lipid species in simple model systems such as yeast', so validate the three-fragment, both-polarity configuration against the reported ~98% accuracy; flag any discrepancy in the output file.

## Related tools

- **LipidCreator** (Generates the target and decoy transition lists and in silico fragment libraries (MS/2 fragment masses for over 60 lipid classes) used to build the background matrix for the probability calculation and the PRM assay.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts the LipidCreator-generated targeted PRM/MRM experiment and processes the resulting target-decoy identification data used for the accuracy ratio.) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe  # start LipidCreator standalone/command-line to build the yeast target-decoy PRM transition list
```

## Evaluation signals

- The computed accuracy ratio at three fragments with both polarities must reproduce the reported ~98% figure (all target lipids identified, only one decoy misidentified) within the article's tolerance; otherwise flag the discrepancy in validation_results.csv.
- Match probabilities must be reported at both tolerances, and the ±2.5 ppm probabilities must be markedly lower than the ±0.5 Da ones, demonstrating the high-resolution filtering effect.
- The probability curve should show a monotonic decrease in random-match probability as fragment count increases from 0 to 3.
- The background matrix must cover the same yeast lipidome species set used to generate the published target-decoy counts, otherwise the comparison is invalid.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates; it supports only knowledge-based selection of transition lists and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order of lipids, and isotope-labeled standards always need to be considered to avoid mis-annotations.
- There are currently no large-scale methods to cope with stereoisomers due to the lack of isotope-coded standards.
- The probability-based identification criterion is validated on simple model systems such as yeast; complex lipidomes may require additional discriminants.

## Evidence

- [discussion] unequivocal identification requires both polarities and multiple fragments: "multiple fragments and both polarity modes are required for unequivocal identification of lipid species in simple model systems such as yeast"
- [discussion] no dedicated FDR scoring: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
- [abstract] fragment mass coverage: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [readme] PRM/MRM scope of the tool: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
