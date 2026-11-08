---
name: annotation-rate-benchmarking
description: Use when you need to report what proportion of chemical features in an
  untargeted LC-MS/MS dataset receive structural annotations from spectral library
  matching versus in silico fingerprint prediction, or to benchmark annotation coverage
  against the Qemistree food-omics reference rates.
when_to_use_negative:
- The dataset's features are already identified with authentic standards at known
  purity — spectral/fingerprint prediction rates are irrelevant as a coverage metric.
- The goal is chemically informed sample comparison (e.g., Weighted UniFrac on a Qemistree
  hierarchy) rather than quantifying annotation coverage.
- Features carry only MS1 data with no MS/MS spectra — Qemistree excludes features
  without MS/MS, so annotation-rate benchmarking cannot be applied.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Qemistree (q2-qemistree)
  role: QIIME 2 plugin whose compute-fragmentation-trees, rerank-molecular-formulas,
    predict-fingerprints, and make-hierarchy commands produce the fingerprint annotations
    and merged feature data used to compute annotation rates
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines molecular formulas and best fragmentation trees per feature, the
    prerequisite for fingerprint prediction
- name: CSI:FingerID
  role: Predicts the 2936 molecular substructure properties from fragmentation trees
    via kernel support vector machines, extending annotation beyond spectral matches
- name: GNPS
  role: Feature-Based Molecular Networking workflow supplies MS2 spectral library
    matches (clusterinfo_summary tsv), the baseline annotation rate
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass,
    direct parent) used for the third annotation rate
- name: MZmine2
  role: Feature detection producing the MGF and feature table inputs from raw MSV000085226
    files
  repo: http://mzmine.github.io
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:38a5e096d6af962e24e346348f4742375fc06fef0e601881c402f83919fcb2a7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  repo_url: https://github.com/biocore/q2-qemistree
  tools:
  - CSI:FingerID
  - ClassyFire
  - GNPS
  - MZmine2
  - QIIME 2
  - Qemistree
  - Qemistree (q2-qemistree)
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# annotation-rate-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies chemical annotation coverage in an untargeted LC-MS/MS metabolomics dataset by computing the fraction of features annotated via GNPS spectral library matches, in silico CSI:FingerID fingerprint predictions (via Qemistree/SIRIUS), and ClassyFire chemical taxonomy, benchmarking against the FoodOmics reference rates (~20% FBMN matches, ~91% fingerprint coverage of 663 quality-filtered features, ~92% ClassyFire assignment).

## When to use

Use when you need to report what proportion of chemical features in an untargeted LC-MS/MS dataset receive structural annotations from spectral library matching versus in silico fingerprint prediction, or to benchmark annotation coverage against the Qemistree food-omics reference rates.

## When NOT to use

- The dataset's features are already identified with authentic standards at known purity — spectral/fingerprint prediction rates are irrelevant as a coverage metric.
- The goal is chemically informed sample comparison (e.g., Weighted UniFrac on a Qemistree hierarchy) rather than quantifying annotation coverage.
- Features carry only MS1 data with no MS/MS spectra — Qemistree excludes features without MS/MS, so annotation-rate benchmarking cannot be applied.

## Inputs

- Raw LC-MS/MS files (.mzXML, .mzML, or .mzDATA) from an untargeted metabolomics run
- MZmine2 outputs: MGF file containing MS1 and MS2 entries, and a feature table of MS1 peak areas per sample (e.g., feature-table.biom)
- GNPS FBMN 'clusterinfo_summary' tsv imported as a QIIME 2 FeatureData[Molecules] artifact (MS2 spectral library matches)

## Outputs

- annotation_rates.csv with three annotation rates: FBMN spectral match fraction (~20%), fingerprint-annotated fraction (~91% of 663 features), ClassyFire-assigned fraction (~92%)
- QIIME 2 artifacts: Phylogeny[Rooted] chemical hierarchy tree and FeatureData[Molecules] with a populated structure_source column
- SiriusFolder artifact (fragmentation trees), Zodiaq/reranked molecular formulas, and CSI:FingerID substructure probability matrix (2936 molecular properties)

## How to apply

Preprocess raw LC-MS/MS files (e.g., from MassIVE accession MSV000085226) with MZmine2 to produce an MGF with MS1+MS2 entries and a feature table of MS1 peak areas, then run GNPS Feature-Based Molecular Networking and import its 'clusterinfo_summary' tsv as FeatureData[Molecules]. Run the q2-qemistree pipeline — compute-fragmentation-trees with --p-ppm-max 15, --p-profile orbitrap, --p-ions-considered '[M+H]+'; rerank-molecular-formulas with --p-zodiac-threshold 0.95; predict-fingerprints with --p-ppm-max 20 for the 2936 molecular properties; make-hierarchy with --i-ms2-matches, which filters MS1 features without fingerprints — to obtain fingerprint-annotated features. Apply get-classyfire-taxonomy to assign the 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent). Then compute three rates to annotation_rates.csv: fraction of features with FBMN spectral library matches (expected ~20%), fraction of quality-filtered features (663 in the food dataset) with structures from GNPS matches plus CSI:FingerID predictions (expected ~91%), and fraction of structures with ClassyFire assignments (expected ~92%). Only features with MS/MS spectra are eligible — features with only MS1 are excluded — and spectral match coverage alone is treated as insufficient (~1–20%) because fingerprint prediction extends coverage to complex samples.

## Related tools

- **Qemistree (q2-qemistree)** (QIIME 2 plugin whose compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, and make-hierarchy commands produce the fingerprint annotations and merged feature data used to compute annotation rates) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines molecular formulas and best fragmentation trees per feature, the prerequisite for fingerprint prediction)
- **CSI:FingerID** (Predicts the 2936 molecular substructure properties from fragmentation trees via kernel support vector machines, extending annotation beyond spectral matches)
- **GNPS** (Feature-Based Molecular Networking workflow supplies MS2 spectral library matches (clusterinfo_summary tsv), the baseline annotation rate)
- **ClassyFire** (Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent) used for the third annotation rate)
- **MZmine2** (Feature detection producing the MGF and feature table inputs from raw MSV000085226 files) — http://mzmine.github.io

## Examples

```
qiime qemistree compute-fragmentation-trees --p-ppm-max 15 --p-profile orbitrap --p-ions-considered '[M+H]+' --i-mgf sirius.mgf --o-fragmentation-trees SiriusFolder
```

## Evaluation signals

- FBMN spectral library match fraction falls within a few percentage points of ~20% on the Global FoodOmics food dataset
- Fingerprint-annotated fraction reaches ~91% of the 663 quality-filtered features when GNPS matches and CSI:FingerID predictions are collated
- ClassyFire assignment rate is ~92% of structures with chemical taxonomy (kingdom/superclass/class/subclass/direct_parent)
- Outputs exist as valid QIIME 2 artifacts: Phylogeny[Rooted] tree and FeatureData[Molecules] with a populated structure_source column
- All annotation rates are written to annotation_rates.csv and features lacking MS/MS (MS1-only) are confirmed absent from the merged feature table

## Limitations

- Typically only 1–20% of features have spectral library matches, depending on how well the sample type has been investigated; fingerprint prediction is needed to raise coverage.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models.
- Only chemical features with MS/MS data are included; features with only MS1 are not considered, and MS1 features without fingerprints are filtered out of the feature table.

## Evidence

- [intro] structural information from chemical features with spectral library matches (typically 1–20% of all features, depending on how well the sample type has been investigated): "structural information from chemical features with spectral library matches (typically 1–20% of all features, depending on how well the sample"
- [intro] SIRIUS determines the molecular formula of each feature using isotope and fragmentation patterns and estimates the best fragmentation tree: "SIRIUS then determines the molecular formula of each feature using the isotope and fragmentation patterns and estimates the best fragmentation tree"
- [methods] MS1 features without fingerprints are filtered out of this feature table.: "MS1 features without fingerprints are filtered out of this feature table."
- [discussion] In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees: "fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees"
- [discussion] While annotations from spectral matches may be more accurate, their coverage is too low to adequately summarize the chemical content of complex samples: "While annotations from spectral matches may be more accurate, their coverage is too low to adequately summarize the chemical content of complex"
