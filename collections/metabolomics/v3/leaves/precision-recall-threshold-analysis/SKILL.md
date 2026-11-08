---
name: precision-recall-threshold-analysis
description: Use when you have LC-MS/MS spectra of samples with known spiked-in standards
  and want to compare multiple spectral-matching scores (e.g. dot product, fraction
  matched, HGS, MVH) by precision/recall across thresholds.
when_to_use_negative:
- Input data are proteomics peptide-spectrum matches or generic machine-learning ranking
  problems rather than metabolite MS/MS spectral-library matching in a metabolomics
  tool like MAVEN2
- You already have a single fixed score cutoff and only need to filter identifications,
  rather than compare several scoring methods across a threshold sweep
- There are no known spiked-in standards available to serve as true positives, so
  precision and recall against ground truth cannot be computed
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAVEN2
  role: Runs the LC-MS/MS spectral search that computes the five candidate match scores
    evaluated in the threshold sweep
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Original open-source LC-MS metabolomics analysis program from which MAVEN2's
    MS/MS spectral matching derives
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:26caaac53e1ff9b270d755b00f16ec84599202b817b4b022a4c85d5c430af0f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - MAVEN
  - MAVEN2
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precision-recall-threshold-analysis

## Summary

Evaluates LC-MS/MS metabolite identification scoring by sweeping score thresholds and computing precision (authentic standards among accepted hits) and recall (standards recovered), as done in MAVEN2's spiked-standards gold standard to show the hypergeometric score (HGS) yields the best precision across thresholds.

## When to use

Use when you have LC-MS/MS spectra of samples with known spiked-in standards and want to compare multiple spectral-matching scores (e.g. dot product, fraction matched, HGS, MVH) by precision/recall across thresholds.

## When NOT to use

- Input data are proteomics peptide-spectrum matches or generic machine-learning ranking problems rather than metabolite MS/MS spectral-library matching in a metabolomics tool like MAVEN2
- You already have a single fixed score cutoff and only need to filter identifications, rather than compare several scoring methods across a threshold sweep
- There are no known spiked-in standards available to serve as true positives, so precision and recall against ground truth cannot be computed

## Inputs

- mzML files of LC-MS/MS runs for spiked-standard samples in multiple background matrices (water and yeast)
- in-house metabolite spectral library (fragmentation spectra released with MAVEN2)
- per-candidate match scores from MAVEN2's LC-MS/MS search module (dot product, fraction matched, fraction of TIC matched, HGS, MVH)
- list of spiked standards defining the true-positive set

## Outputs

- precision_recall_scores.csv containing precision and recall values for all five scoring methods at multiple thresholds, separated by background
- summary of which scoring method attains the best precision across thresholds

## How to apply

Load the mzML files for the spiked-standard samples (e.g. water-background and yeast-background series) into MAVEN2's LC-MS/MS search module and search each file against the released in-house metabolite spectral library, computing every candidate identification's match scores: dot product, fraction matched, fraction of TIC matched, highest grouped score (HGS), and MVH. Treat the spiked standards as true positives and rank all library matches independently by each of the five scores. Sweep the score threshold across its range for each scoring method, computing at each threshold the precision (fraction of accepted identifications that are authentic standards) and recall (fraction of spiked standards recovered), separately for each background matrix. Tabulate the precision/recall points for all methods and backgrounds and identify which score maintains the best precision over the widest threshold range; in the MAVEN2 evaluation this was the hypergeometric score, especially in the more complex yeast background.

## Related tools

- **MAVEN2** (Runs the LC-MS/MS spectral search that computes the five candidate match scores evaluated in the threshold sweep) — https://github.com/eugenemel/maven
- **MAVEN** (Original open-source LC-MS metabolomics analysis program from which MAVEN2's MS/MS spectral matching derives) — https://github.com/eugenemel/maven

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven && cd maven && qmake -r build.pro && make -j4
```

## Evaluation signals

- precision_recall_scores.csv contains precision and recall values for all five scoring methods (dot product, fraction matched, fraction of TIC matched, HGS, MVH) at multiple thresholds
- Precision and recall are computed separately for the water-background and yeast-background sample series
- The comparison reproduces the paper's finding that the hypergeometric score (HGS) achieves the best precision across a large range of thresholds, especially in the yeast background
- Spiked standards are consistently counted as true positives; recall equals the fraction of spiked standards recovered at each threshold
- Every accepted-identification set used for precision comes from ranking library matches by a single score, so each method's curve is independent of the others

## Limitations

- Precision at a given threshold is background-dependent: the more complex yeast background is harder than water, so results do not transfer to matrices of different complexity
- Scores are specific to MAVEN2's search implementation; thresholds derived here cannot be reused directly for other tools' score scales
- The gold standard only covers the standards included in the spiked set; identifications of compounds absent from the spike list are treated as spurious regardless of biological reality

## Evidence

- [abstract] We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds.: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [readme] Scripts to create in silico lipid fragmentation spectra, based on analysis of chemical standards and relevant scientific literature.: [evidence span withheld — anchor status: external_doc]
