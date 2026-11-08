---
name: lipid-fragment-mass-probability-computation
description: 'Use when designing or auditing a targeted lipidomics PRM/MRM assay:
  you must decide how many MS/MS fragments (and which tolerance, e.g. ±0.5 Da vs ±2.5
  ppm) are needed so that target lipids cannot be confused with decoys within the
  sample lipidome background.'
when_to_use_negative:
- You need an automated FDR scoring algorithm — LipidCreator does not provide a dedicated
  scoring algorithm to determine false discovery rates and intentionally does not
  address coeluting/isobaric lipids and peak area integration.
- Your analysis is untargeted lipidome-wide screening rather than targeted assay development
  for a defined set of lipids of interest.
- Your sample requires stereoisomer resolution — the article notes no large-scale
  methods exist to cope with stereoisomers due to lack of isotope-coded standards.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: LipidCreator
  role: Computes MS/MS fragment masses for >60 lipid classes and generates target
    lists and in silico spectral libraries; supports standalone and command-line operation
    for the background-matching probability calculation.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts LipidCreator as a plugin to build the PRM transition lists and target-decoy
    assays whose identification counts feed the accuracy comparison.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:ea1019bb777a2070afc33ebb00c886a352e9b0befe572fa258846a9bdee9e1dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-fragment-mass-probability-computation

## Summary

Computes, per fragment count (0–3) and mass tolerance (±0.5 Da vs ±2.5 ppm), the probability that a precursor–fragment pair identifies a lipid species unambiguously against a background lipidome, and validates the resulting accuracy against target-decoy PRM identification counts (e.g. ~98% for yeast with three fragments in both polarities).

## When to use

Use when designing or auditing a targeted lipidomics PRM/MRM assay: you must decide how many MS/MS fragments (and which tolerance, e.g. ±0.5 Da vs ±2.5 ppm) are needed so that target lipids cannot be confused with decoys within the sample lipidome background.

## When NOT to use

- You need an automated FDR scoring algorithm — LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Your analysis is untargeted lipidome-wide screening rather than targeted assay development for a defined set of lipids of interest.
- Your sample requires stereoisomer resolution — the article notes no large-scale methods exist to cope with stereoisomers due to lack of isotope-coded standards.

## Inputs

- Background lipidome species list (e.g. yeast lipidome) as the candidate matrix
- Enumerated lipid species fragment masses (LipidCreator-computed MS/MS fragment masses per species)
- Mass tolerance settings (±0.5 Da coarse, ±2.5 ppm high-resolution)
- Published target-decoy LC/ESI PRM identification counts (true-target vs false-decoy)

## Outputs

- Fragment-count-dependent ambiguity probabilities at ±0.5 Da and ±2.5 ppm
- Accuracy-ratio comparison (computed vs reported, e.g. 98% at three fragments)
- validation_results.csv with probabilities, accuracy ratios, and any flagged discrepancies

## How to apply

Load the background lipidome species list (e.g. the yeast lipidome) and enumerate candidate lipid species and their fragment masses, which LipidCreator can compute for over 60 lipid classes. For each fragment count (0, 1, 2, 3), compute the probability that a random precursor–fragment pair matches unambiguously within the background at ±0.5 Da tolerance, then repeat at ±2.5 ppm to quantify how high-resolution filtering collapses the ambiguity probability. Separately, take the published target-decoy LC/ESI-PRM validation counts (true-target vs false-decoy identifications) and derive the accuracy ratio — for the yeast lipidome this is all targets identified with only one decoy misidentified, i.e. 98% accuracy with three fragments across positive and negative polarity modes. Compare computed probabilities against the reported accuracy ratios and write fragment-count-dependent probabilities plus the accuracy comparison into validation_results.csv. Rationale: the paper establishes that multiple fragments and both polarity modes are required for unequivocal identification in simple model systems such as yeast, and LipidCreator deliberately supplies a knowledge-based transition selection rather than a scoring algorithm, so ambiguity must be quantified by this background-matching probability calculation.

## Related tools

- **LipidCreator** (Computes MS/MS fragment masses for >60 lipid classes and generates target lists and in silico spectral libraries; supports standalone and command-line operation for the background-matching probability calculation.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts LipidCreator as a plugin to build the PRM transition lists and target-decoy assays whose identification counts feed the accuracy comparison.) — https://skyline.ms/project/home/software/Skyline/begin.view

## Evaluation signals

- At three fragments across both positive and negative polarity modes, the computed accuracy ratio reproduces the reported ~98% figure for the yeast target-decoy LC-ESI PRM assay; otherwise flag the discrepancy in validation_results.csv.
- The ±2.5 ppm probability computation yields strictly lower ambiguity probabilities than the ±0.5 Da computation for every fragment count, quantifying high-resolution filtering.
- All target lipids are identified and exactly one decoy is misidentified in the yeast reference counts, matching the reported result.
- The fragment-count series (0, 1, 2, 3) is monotonic, with ambiguity probability decreasing as fragments are added.
- Every enumerated fragment mass falls within the >60 lipid classes LipidCreator supports.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates; it intentionally does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order, and isotope labeled standards must be considered to avoid mis-annotations.
- No large-scale methods are available to cope with stereoisomers due to the lack of isotope-coded standards; the current implementation focuses on eukaryotic model systems due to lack of standards for bacteria such as E. coli and archaea.

## Evidence

- [discussion] Unequivocal yeast identification requires multiple fragments and both polarities: "multiple fragments and both polarity modes are required for unequivocal identification of lipid species in simple model systems such as yeast"
- [abstract] Fragment mass computation coverage: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [readme] Assay construction role of LipidCreator/Skyline: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [discussion] No built-in FDR scoring: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
