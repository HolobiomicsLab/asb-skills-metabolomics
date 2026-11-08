---
name: matrix-background-effect-assessment
description: Use when evaluating whether sample matrix complexity (e.g., water vs
  yeast background) degrades MS/MS identification precision, by comparing spectral-match
  score thresholds across matrix conditions.
when_to_use_negative:
- Input is already a feature/peak table without MS/MS spectra, so spectral-library
  match scores cannot be computed
- No experimentally spiked authentic standards are available, so true positives and
  precision cannot be defined
- The goal is generating an in-silico lipid fragmentation library (e.g., with CalicoLipidLibrary)
  rather than evaluating matrix effects on identifications
edam_operation: http://edamontology.org/operation_3214
tools:
- name: MAVEN2
  role: Performs the LC-MS/MS spectral library search and computes the five match
    scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) used
    to build precision/recall curves per matrix background
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Predecessor open-source LC-MS metabolomics analysis program from which MAVEN2's
    spectral matching evaluation workflow derives
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Companion package that generates in-silico lipid fragmentation spectra (MSP
    libraries) that can serve as search libraries for spectral matching evaluations
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:431d5cfc7eeb14cabb83b63d7d4c673f53af77aa366ffb44b5e7c9b309aae702
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN
  - MAVEN2
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# matrix-background-effect-assessment

## Summary

Assesses how sample matrix complexity affects MS/MS metabolite identification precision by comparing score-threshold precision/recall curves from MAVEN2 spectral library searches on standards spiked into water versus yeast backgrounds.

## When to use

Use when evaluating whether sample matrix complexity (e.g., water vs yeast background) degrades MS/MS identification precision, by comparing spectral-match score thresholds across matrix conditions.

## When NOT to use

- Input is already a feature/peak table without MS/MS spectra, so spectral-library match scores cannot be computed
- No experimentally spiked authentic standards are available, so true positives and precision cannot be defined
- The goal is generating an in-silico lipid fragmentation library (e.g., with CalicoLipidLibrary) rather than evaluating matrix effects on identifications

## Inputs

- mzML files of spiked-standard samples in a water background
- mzML files of spiked-standard samples in a yeast (complex matrix) background
- in-house metabolite MS/MS spectral library searched by MAVEN2
- list of authentic spiked standards used as true positives

## Outputs

- precision/recall values for all five scoring methods (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple score thresholds
- precision_recall_scores.csv with points for both water and yeast backgrounds
- summary of which method attains best precision across thresholds per matrix

## How to apply

Load the mzML files for water-background and yeast-background spiked-standard samples into MAVEN2's LC-MS/MS search module and search each file against the in-house metabolite spectral library, computing all five MAVEN2 match scores for every candidate identification: dot product, fraction matched, fraction of TIC matched, highest grouped score (HGS), and MVH. Treat the spiked standards as true positives and, for each scoring method and each background separately, sweep the score threshold across its range, computing precision (fraction of accepted identifications that are authentic standards) and recall (fraction of spiked standards recovered) at each threshold. Tabulate the precision/recall points for all five methods and both backgrounds (e.g., to precision_recall_scores.csv) and compare the water and yeast curves to quantify how the more complex matrix affects separation of authentic from spurious identifications. The decision point is which method retains best precision as the matrix gets more complex; in this evaluation the hypergeometric score (HGS) achieved the best precision across a large range of thresholds, especially in the yeast background.

## Related tools

- **MAVEN2** (Performs the LC-MS/MS spectral library search and computes the five match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) used to build precision/recall curves per matrix background) — https://github.com/eugenemel/maven
- **MAVEN** (Predecessor open-source LC-MS metabolomics analysis program from which MAVEN2's spectral matching evaluation workflow derives) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Companion package that generates in-silico lipid fragmentation spectra (MSP libraries) that can serve as search libraries for spectral matching evaluations) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven && cd maven && qmake -r build.pro && make -j4
```

## Evaluation signals

- Output contains precision and recall values for all five scoring methods (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple thresholds
- Separate precision/recall curves exist for the water and yeast backgrounds so matrix-complexity effects can be compared directly
- The comparison reproduces the paper's reported result that the hypergeometric score (HGS) yields the best precision across thresholds, most pronounced in the yeast background
- Recall approaches 1.0 (all spiked standards recovered) at low thresholds while precision degrades as thresholds are relaxed
- Spiked standards are consistently counted as true positives across all five scoring methods and both matrices

## Limitations

- Matrix effects were only characterized for water and yeast backgrounds; other matrix types are not covered by this evaluation
- Requires an in-house spectral library and authentic standards; without them precision cannot be computed
- MAVEN2 must be built from source via recursive clone (including maven_core submodules) with qmake/make; Linux builds have been retired and mac/windows executables are produced by Appveyor
- Repository changelog is incomplete for some releases, so version-to-version behavioral differences may be undocumented

## Evidence

- [abstract] We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds.: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [results] Among the scoring methods evaluated, the hypergeometric score achieved the best precision across a large range of score thresholds, especially in the more complex yeast background samples.: [evidence span withheld — anchor status: not_found]
- [readme] As of 20241105, linux builds have been retired, and both mac os and windows executable are now produced by Appveyor.: "As of 20241105, linux builds have been retired, and both mac os and windows executable are now produced by Appveyor."
