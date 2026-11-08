---
name: metabolite-annotation-scoring
description: Use when mzML files from spiked-standard experiments (e.g., standards
  in water or yeast backgrounds) have been searched against a spectral library and
  you must rank annotations by score and select the -performing scoring method and
  threshold.
when_to_use_negative:
- The goal is in-silico lipidomics library generation or comparison (e.g., CalicoLipidLibrary
  MSP generation) rather than empirical precision evaluation of match scores
- Input data lack a gold-standard set of authentic standards, so precision and recall
  cannot be computed
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MAVEN2
  role: Performs LC-MS/MS spectral matching against the metabolite spectral library
    and computes the five match scores (dot product, fraction matched, fraction of
    TIC matched, HGS, MVH) for each candidate identification
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Original open-source LC-MS metabolomics platform from which MAVEN2's LC-MS/MS
    search module is derived
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:df1f3bd5b00eca7d0c5ef5c4ee75e1b31bd76f68586ba39c55c623c06ea15268
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN
  - MAVEN2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-annotation-scoring

## Summary

Evaluate MS/MS spectral-matching identifications by sweeping the score threshold for each of MAVEN2's five match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) and computing precision/recall against a spiked-standards gold standard, to determine which score best separates authentic from spurious identifications.

## When to use

Use when mzML files from spiked-standard experiments (e.g., standards in water or yeast backgrounds) have been searched against a spectral library and you must rank annotations by score and select the best-performing scoring method and threshold.

## When NOT to use

- The goal is in-silico lipidomics library generation or comparison (e.g., CalicoLipidLibrary MSP generation) rather than empirical precision evaluation of match scores
- Input data lack a gold-standard set of authentic standards, so precision and recall cannot be computed

## Inputs

- mzML files of LC-MS/MS runs of standards spiked into water background and into yeast background
- in-house metabolite spectral library (released MAVEN2 library) searched against each file
- list of spiked standards defining the true-positive set

## Outputs

- precision_recall_scores.csv with precision and recall values for all five scoring methods (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple thresholds
- summary statement of which scoring method achieves the best precision across thresholds

## How to apply

Load the mzML files into MAVEN2's LC-MS/MS search module and search against the in-house metabolite spectral library, computing all five candidate match scores (dot product, fraction matched, fraction of TIC matched, highest grouped score HGS, and MVH) for every candidate identification. Treat the spiked standards as true positives and rank all library matches independently by each of the five scores. Sweep each score's threshold across its range, computing precision (fraction of accepted identifications that are authentic standards) and recall (fraction of spiked standards recovered) at every threshold, separately for the water and yeast backgrounds. Tabulate all precision/recall points in precision_recall_scores.csv and summarize which method attains the best precision across thresholds; the paper's validation expectation is that HGS yields the best precision across a large range of thresholds, especially in the complex yeast background.

## Related tools

- **MAVEN2** (Performs LC-MS/MS spectral matching against the metabolite spectral library and computes the five match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) for each candidate identification) — https://github.com/eugenemel/maven
- **MAVEN** (Original open-source LC-MS metabolomics platform from which MAVEN2's LC-MS/MS search module is derived) — https://github.com/eugenemel/maven

## Evaluation signals

- precision_recall_scores.csv contains precision and recall values for all five scoring methods at multiple thresholds, for both water and yeast backgrounds
- The threshold sweep reproduces the paper's result that HGS achieves the best precision across a large range of score thresholds
- The performance gap favoring HGS is most pronounced in the more complex yeast background samples
- At each threshold point, precision equals the fraction of accepted identifications that are authentic spiked standards and recall equals the fraction of spiked standards recovered

## Limitations

- The evaluation depends on the specific in-house metabolite spectral library and spiked-standard set; conclusions may not transfer to other libraries or matrix backgrounds
- Ranking is done per score independently, so scores are compared only through their precision/recall profiles, not on a common calibrated scale

## Evidence

- [abstract] We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds.: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] MAVEN2 supports LC-MS/MS analysis of metabolomics and lipidomics samples: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [abstract] MAVEN2 source code and installers are freely available on GitHub: "MAVEN2 source code and cross-platform application installers are freely available for download from GitHub under a GNU permissive license [ver 3]"
- [readme] Download latest release from Maven Releases on GitHub: "## Download latest release

[Maven Releases](https://github.com/eugenemel/maven/releases)"
