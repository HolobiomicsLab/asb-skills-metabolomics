---
name: metabolite-annotation-mapping
description: 'Use when untargeted LC-MS/MS features (from MZmine2 output: feature
  table + MGF spectra) need molecular formulas, fingerprints, and chemical-taxonomy
  annotations attached to a chemical tree so annotated molecules can be traced across
  samples.'
when_to_use_negative:
- The dataset contains only MS1 features without MS/MS spectra — fingerprints cannot
  be predicted and such features are filtered out.
- You need complete chemical characterization of a complex sample — spectral match
  annotation coverage is typically only 1–20% of features, too low to adequately summarize
  chemical content.
- Comparison of already co-processed datasets where retention-time alignment is valid;
  the fingerprint-based representation is mainly advantageous when repeating feature
  detection/alignment is impractical.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: q2-qemistree
  role: QIIME 2 plugin that runs SIRIUS fragmentation-tree computation, fingerprint
    prediction, hierarchy construction, and ClassyFire taxonomy assignment
  repo: https://github.com/biocore/q2-qemistree
- name: SIRIUS
  role: Determines molecular formula from isotope/fragmentation patterns and estimates
    the best fragmentation tree for each feature
- name: CSI:FingerID
  role: Predicts 2936 molecular properties from fragmentation trees using kernel support
    vector machines
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy to annotated molecules
- name: QIIME 2
  role: Hosts the q2-qemistree plugin and provides diversity metrics such as Weighted
    UniFrac on the chemical hierarchy
- name: Empress
  role: Interactive visualization of the Qemistree tree with sample/feature metadata
    coloring
  repo: https://github.com/biocore/empress
- name: GNPS
  role: Web-based mass spectrometry infrastructure providing spectral library matches
    that feed annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:5e881aa333c75c6d203f593456d199e8cb048673120d0f2fc1eb53f7d3f6c153
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
  - Empress
  - GNPS
  - QIIME 2
  - Qemistree
  - SIRIUS
  - q2-qemistree
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7407407407407407
n_claims: 27
---

# metabolite-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps metabolite annotations — spectral library matches and in silico properties predicted by SIRIUS/CSI:FingerID — onto the Qemistree chemical hierarchy, assigning each MS/MS feature a ClassyFire 5-level chemical taxonomy so annotations can be interpreted in the context of sample metadata.

## When to use

Use when untargeted LC-MS/MS features (from MZmine2 output: feature table + MGF spectra) need molecular formulas, fingerprints, and chemical-taxonomy annotations attached to a chemical tree so annotated molecules can be traced across samples.

## When NOT to use

- The dataset contains only MS1 features without MS/MS spectra — fingerprints cannot be predicted and such features are filtered out.
- You need complete chemical characterization of a complex sample — spectral match annotation coverage is typically only 1–20% of features, too low to adequately summarize chemical content.
- Comparison of already co-processed datasets where retention-time alignment is valid; the fingerprint-based representation is mainly advantageous when repeating feature detection/alignment is impractical.

## Inputs

- LC-MS/MS feature table with observed ions and peak areas (feature-table.biom)
- MS/MS fragmentation spectra in MGF format (e.g., sirius.mgf from MZmine2 preprocessing of .mzXML/.mzML/.mzDATA files)
- SIRIUS fragmentation trees / predicted molecular fingerprints per feature
- Spectral library match annotations (e.g., from GNPS) for ClassyFire taxonomy assignment
- Sample metadata mapping samples to sources (e.g., food type TSV)

## Outputs

- Feature metadata table mapping each feature to molecular formula, fingerprint, and 5-level ClassyFire taxonomy (chemical kingdom, superclass, class, subclass, direct parent)
- Pruned chemical hierarchy (tree of MS1 features retaining only annotated molecules)
- Sample-to-sample chemical distances (e.g., Weighted UniFrac on the chemical tree)
- Presence/absence tables of ingredient-derived signals (e.g., phosphoethanolamine in egg scramble) exported as CSV

## How to apply

First perform feature detection on .mzXML/.mzML/.mzDATA files with MZmine2 to produce a feature table (peak areas) and an MGF of MS/MS spectra; only chemical features with MS/MS data are eligible, features with only MS1 are excluded. Run SIRIUS to determine each feature's molecular formula from isotope and fragmentation patterns and estimate the best fragmentation tree, then CSI:FingerID predicts 2936 molecular properties from those fragmentation trees via kernel support vector machines. Build the hierarchy from fingerprint vectors (optionally adding --p-qc-properties to keep only the 489 PubChem fingerprint positions), then use ClassyFire to assign a 5-level taxonomy (chemical kingdom, superclass, class, subclass, direct parent ontology) to all molecules annotated via spectral library matching. Prune the molecular hierarchy to keep only molecules with annotations, and filter MS1 features without fingerprints out of the feature table. The annotated hierarchy plus sample metadata can then drive chemically informed comparisons (e.g., Weighted UniFrac via QIIME 2) and visualization with q2-empress, as in the FoodOmics analysis tracing egg-derived phosphoethanolamine into the egg scramble.

## Related tools

- **q2-qemistree** (QIIME 2 plugin that runs SIRIUS fragmentation-tree computation, fingerprint prediction, hierarchy construction, and ClassyFire taxonomy assignment) — https://github.com/biocore/q2-qemistree
- **SIRIUS** (Determines molecular formula from isotope/fragmentation patterns and estimates the best fragmentation tree for each feature)
- **CSI:FingerID** (Predicts 2936 molecular properties from fragmentation trees using kernel support vector machines)
- **ClassyFire** (Assigns the 5-level chemical taxonomy to annotated molecules)
- **QIIME 2** (Hosts the q2-qemistree plugin and provides diversity metrics such as Weighted UniFrac on the chemical hierarchy)
- **Empress** (Interactive visualization of the Qemistree tree with sample/feature metadata coloring) — https://github.com/biocore/empress
- **GNPS** (Web-based mass spectrometry infrastructure providing spectral library matches that feed annotation)

## Examples

```
qiime qemistree get-classyfire-taxonomy --help
```

## Evaluation signals

- Every feature carrying a taxonomy assignment has MS/MS data; MS1-only features appear in neither the annotation table nor the pruned hierarchy.
- With --p-qc-properties applied, the fingerprint contingency table contains only the 489 PubChem fingerprint positions.
- The pruned hierarchy contains exactly the molecules with annotations — no unannotated tips remain.
- Downstream sanity check: Weighted UniFrac on the hierarchy should recover known relationships, e.g., the phosphoethanolamine signal present in egg scramble but absent in the other highlighted complex foods.
- Annotation coverage falls within the reported 1–20% range of features with spectral library matches; near-100% coverage signals a pipeline error.

## Limitations

- Typically only 1–20% of features have spectral library matches, depending on how well the sample type has been investigated.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases used to train the predictive models; Qemistree shares limitations with CSI:FingerID.
- In negative ionization mode, fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra, resulting in less-extensive trees.
- Spectral match annotations alone have coverage too low to adequately summarize the chemical content of complex samples.

## Evidence
