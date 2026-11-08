---
name: spectral-library-matching
description: Use when an MGF with MS1+MS2 entries and a feature table from MZmine2
  need MS2 library annotations, or when quantifying the fraction of features with
  spectral library matches (typically 1–20%).
when_to_use_negative:
- Features lack MS/MS data — only chemical features with MS2 spectra are considered;
  MS1-only features are not matchable
- The goal is de novo formula and fingerprint prediction without reference spectra
  — use the SIRIUS/CSI:FingerID fragmentation-tree steps instead
- Feature detection (MZmine2) has not yet been run and no MGF/feature table exists
  — spectral matching is a downstream step, not a replacement
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: GNPS
  role: Hosts the Feature-Based Molecular Networking workflow that produces MS2 spectral
    library matches and the clusterinfo_summary tsv
- name: Qemistree
  role: make-hierarchy consumes the MS2 matches (--i-ms2-matches) to merge library
    annotations with fingerprint predictions into combined feature data
  repo: https://github.com/biocore/q2-qemistree
- name: QIIME 2
  role: Artifact framework into which the clusterinfo_summary tsv is imported as FeatureData[Molecules]
  repo: https://github.com/biocore/q2-qemistree
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy to molecules annotated via spectral
    library matching and in silico prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:c9606c36aedcaf57b923a8808eeec9fd2a840d78be667764fa6e1e00e4c4dffc
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
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates LC-MS/MS chemical features by matching their MS2 fragmentation spectra against reference spectral libraries via GNPS Feature-Based Molecular Networking (FBMN), yielding molecular identifiers that Qemistree merges with in silico CSI:FingerID predictions for taxonomy and tree building.

## When to use

Use when an MGF with MS1+MS2 entries and a feature table from MZmine2 need MS2 library annotations, or when quantifying the fraction of features with spectral library matches (typically 1–20%).

## When NOT to use

- Features lack MS/MS data — only chemical features with MS2 spectra are considered; MS1-only features are not matchable
- The goal is de novo formula and fingerprint prediction without reference spectra — use the SIRIUS/CSI:FingerID fragmentation-tree steps instead
- Feature detection (MZmine2) has not yet been run and no MGF/feature table exists — spectral matching is a downstream step, not a replacement

## Inputs

- MGF file with MS1 and MS2 entries from MZmine2 feature detection
- Feature table of MS1 peak areas per sample (BIOM/quant table)
- GNPS FBMN output 'clusterinfo_summary' tsv of MS2 spectral library matches

## Outputs

- QIIME 2 FeatureData[Molecules] artifact with spectral library match annotations
- Merged feature data combining library matches and CSI:FingerID predictions with structure_source column populated
- Annotation-rate summary (e.g., annotation_rates.csv) reporting the fraction of features with FBMN matches (~20%)

## How to apply

Run feature detection in MZmine2 (.mzXML/.mzML inputs) to produce an MGF containing MS1 and MS2 entries plus a feature table of MS1 peak areas, then submit to the GNPS FBMN workflow to obtain MS2 spectral library matches. Import the GNPS 'clusterinfo_summary' tsv into QIIME 2 as a FeatureData[Molecules] artifact. Pass this artifact as --i-ms2-matches to qiime qemistree make-hierarchy so library matches are merged with CSI:FingerID substructure probabilities (2936 molecular properties) into combined feature data; only features with fingerprints survive this merge. Run qiime qemistree get-classyfire-taxonomy to assign kingdom/superclass/class/subclass/direct_parent ontology to matched and predicted molecules. Report the library-match rate as the fraction of features with FBMN spectral library matches — in the Global FoodOmics food dataset this was ~20%, far below the ~91% coverage once CSI:FingerID predictions were collated with matches across 663 quality-filtered features. Rationale: spectral matches are the most accurate annotations but their coverage is too low alone to summarize complex samples, so they anchor the fingerprint-based hierarchy rather than serving as the sole annotation source.

## Related tools

- **GNPS** (Hosts the Feature-Based Molecular Networking workflow that produces MS2 spectral library matches and the clusterinfo_summary tsv)
- **Qemistree** (make-hierarchy consumes the MS2 matches (--i-ms2-matches) to merge library annotations with fingerprint predictions into combined feature data) — https://github.com/biocore/q2-qemistree
- **QIIME 2** (Artifact framework into which the clusterinfo_summary tsv is imported as FeatureData[Molecules]) — https://github.com/biocore/q2-qemistree
- **ClassyFire** (Assigns the 5-level chemical taxonomy to molecules annotated via spectral library matching and in silico prediction)

## Examples

```
qiime tools import --input-path clusterinfo_summary.tsv --type 'FeatureData[Molecules]' --output-path ms2-matches.qza
```

## Evaluation signals

- The FBMN run returns a clusterinfo_summary tsv that imports cleanly as a QIIME 2 FeatureData[Molecules] artifact
- Computed library-match rate falls within a few percentage points of the reported ~20% for the Global FoodOmics food dataset
- Merged feature data has a populated structure_source column distinguishing GNPS library matches from CSI:FingerID predictions
- Features without MS2 spectra are excluded from matching (only chemical features with MS/MS data are considered)
- MS2-matched features are retained in the merged tree/table after make-hierarchy filters MS1 features without fingerprints

## Limitations

- Spectral library match coverage is typically only 1–20% of all features, depending on how well the sample type has been investigated
- Annotations from spectral matches alone are too low-coverage to adequately summarize the chemical content of complex samples
- In negative ionization mode, fewer confident annotations/fingerprints are possible due to fewer publicly available reference spectra, yielding less-extensive trees

## Evidence
