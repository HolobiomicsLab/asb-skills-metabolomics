---
name: spiked-standard-ground-truth-evaluation
description: Use when a spiked-standards sample (authentic metabolites in water or
  a complex yeast background, e.g. mzML) is available as ground truth and candidate
  spectral-match scores such as dot product, fraction matched, fraction of TIC matched,
  MVH or HGS must be ranked by precision/recall.
when_to_use_negative:
- The samples contain no spiked authentic standards, so no ground-truth labeling of
  true positives is possible and precision cannot be computed.
- The goal is a routine metabolite identification run rather than an evaluation/comparison
  of scoring schemes against a gold standard.
- The spectra are lipidomics searches against the in-silico CalicoLipidLibrary rather
  than searched against an experimental metabolite spectral library with spiked standards.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: MAVEN2
  role: Runs the LC-MS/MS spectral library search on the spiked-sample mzML files
    and computes the five match scores (dot product, fraction matched, fraction of
    TIC matched, HGS, MVH) per candidate identification
  repo: https://github.com/eugenemel/maven
- name: maven_core
  role: Supporting library compiled and used by MAVEN2 for the metabolomic analysis
    underlying the evaluation
  repo: https://github.com/eugenemel/maven_core
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:7c4dbd68d34650c61b75f23dfab903745bc8f4737f396dcfae83f17036cd2127
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN2
  - maven_core
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spiked-standard-ground-truth-evaluation

## Summary

Builds a precision/recall gold-standard evaluation of MS/MS spectral matching scores by treating metabolite standards spiked into water and yeast backgrounds as true positives and sweeping each score threshold. In the MAVEN2 study it showed the hypergeometric score (HGS) achieved the best precision across thresholds, especially in the yeast background.

## When to use

Use when a spiked-standards sample (authentic metabolites in water or a complex yeast background, e.g. mzML) is available as ground truth and candidate spectral-match scores such as dot product, fraction matched, fraction of TIC matched, MVH or HGS must be ranked by precision/recall.

## When NOT to use

- The samples contain no spiked authentic standards, so no ground-truth labeling of true positives is possible and precision cannot be computed.
- The goal is a routine metabolite identification run rather than an evaluation/comparison of scoring schemes against a gold standard.
- The spectra are lipidomics searches against the in-silico CalicoLipidLibrary rather than searched against an experimental metabolite spectral library with spiked standards.

## Inputs

- mzML files of spiked-standard samples in a simple water background
- mzML files of the same spiked standards in a complex yeast background
- in-house metabolite MS/MS spectral library (spectral library searched by MAVEN2)
- list of spiked standard compounds defining true positives

## Outputs

- precision/recall values for each of the five scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple thresholds and both backgrounds
- a tabulated comparison file such as precision_recall_scores.csv
- a summary identifying which scoring method attains the best precision across thresholds

## How to apply

Load the mzML files for the water-background and yeast-background spiked-standard samples into MAVEN2's LC-MS/MS search module and search them against the in-house metabolite spectral library, which computes five match scores per candidate identification: dot product, fraction matched, fraction of TIC matched, highest grouped score (HGS), and MVH. Treat all matches to spiked standards as true positives and rank every library hit independently under each of the five scores. Sweep each score's threshold over its range and, at every threshold, record precision (fraction of accepted identifications that are authentic standards) and recall (fraction of spiked standards recovered), separately per background. Tabulate all methods and both backgrounds (e.g. into precision_recall_scores.csv) and compare precision-versus-threshold curves; the method whose precision stays highest across the widest threshold range wins. The yeast background is the harder test because the complex matrix generates more spurious identifications, so it best discriminates between scoring schemes.

## Related tools

- **MAVEN2** (Runs the LC-MS/MS spectral library search on the spiked-sample mzML files and computes the five match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) per candidate identification) — https://github.com/eugenemel/maven
- **maven_core** (Supporting library compiled and used by MAVEN2 for the metabolomic analysis underlying the evaluation) — https://github.com/eugenemel/maven_core

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- The output contains precision and recall values for all five scoring methods (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple thresholds.
- Precision and recall are reported separately for both the water and yeast backgrounds.
- All matches to spiked standards are consistently labeled as true positives across every score ranking.
- The comparison reproduces the paper's reported result that the HGS (hypergeometric) score yields the best precision across a large range of score thresholds, especially in yeast background samples.
- Precision declines and recall increases monotonically as the score threshold is lowered for each method.

## Limitations

- Evaluation depends on the specific in-house spectral library used; conclusions may not transfer to other libraries such as the in-silico lipidomics libraries.
- Only the yeast and water backgrounds were tested; other matrix complexities are unvalidated.
- Building the tool from source can be environment-sensitive (the maven_core README documents an MSYS2 package bug and retired Linux builds, with macOS and Windows executables produced by Appveyor).

## Evidence

- [abstract] Abstract: ability of scoring to separate authentic from spurious identifications via spiked standards: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] Abstract: algorithms for MS/MS spectral matching: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [methods] Methods: building MAVEN2 from source: "git clone --recursive  git@github.com:eugenemel/maven.git maven
cd maven
qmake -r build.pro
make -j4"
- [readme] README: tool identity: "## Maven GUI: Metabolomics Analysis and Visualization Engine"
