---
name: peptidoform-prediction-feature-annotation
description: Use when raw search-engine PSM identifications (e.g. mzIdentML from X!Tandem,
  MaxQuant, or PEAKS) must be enriched with predicted-peptidoform features before
  semi-supervised rescoring, especially for non-tryptic immunopeptidomics data.
when_to_use_negative:
- Input is already a post-rescoring feature table or scored PSM output — no further
  peptidoform prediction features are needed
- Data lacks LC-MS context (e.g. no retention time or fragment spectra available)
  so MS2PIP/DeepLC features cannot be computed
- The goal is tool comparison/availability review (e.g. reproducing Table 1 of the
  rescoring-pipeline comparison) rather than actual feature annotation
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MS2Rescore
  role: Orchestrates MS2PIP and DeepLC feature generation and modification parsing
    to ProForma for rescoring input
  repo: https://github.com/compomics/ms2rescore
- name: MS2PIP
  role: Predicts MS2 fragment ion intensities used as rescoring features (HCD model
    by default)
  repo: https://github.com/compomics/ms2rescore
- name: DeepLC
  role: Predicts retention time features from peptidoform sequences
  repo: https://github.com/compomics/ms2rescore
- name: NetMHCpan
  role: Predicted MHC binding affinity source usable as an affinity-based rescoring
    feature (supported by MHCquant and inSPIRE)
- name: Prosit
  role: Deep-learning predictor of peptidoform properties (intensities, retention
    time, collisional cross section) referenced in the review
- name: Mumble
  role: PSM generator that expands peptidoforms with Unimod candidate modifications
    before feature annotation
  repo: https://github.com/compomics/mumble
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:71699b8e7ef3ae1ae7361263067aa9d9708f8f20169b95290349fa8a5ec82bcf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - AlphaPeptDeep
  - DeepLC
  - DeepRescore
  - INFERYS
  - MHCquant
  - MS2PIP
  - MS2Rescore
  - MSBooster
  - Mumble
  - NetMHCpan
  - Oktoberfest
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# peptidoform-prediction-feature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate peptide-spectrum matches with features derived from deep-learning predictors of peptidoform properties (MS2 fragment ion intensities, retention time, collisional cross section) so that semi-supervised PSM rescoring can better separate correct from incorrect immunopeptide identifications.

## When to use

Use when raw search-engine PSM identifications (e.g. mzIdentML from X!Tandem, MaxQuant, or PEAKS) must be enriched with predicted-peptidoform features before semi-supervised rescoring, especially for non-tryptic immunopeptidomics data.

## When NOT to use

- Input is already a post-rescoring feature table or scored PSM output — no further peptidoform prediction features are needed
- Data lacks LC-MS context (e.g. no retention time or fragment spectra available) so MS2PIP/DeepLC features cannot be computed
- The goal is tool comparison/availability review (e.g. reproducing Table 1 of the rescoring-pipeline comparison) rather than actual feature annotation

## Inputs

- Search-engine PSM files (e.g. mzIdentML, TSV) with unfiltered spectrum annotations
- Raw or centroided spectrum files linked by spectrum_id_pattern
- Modification mapping configuration mapping search-engine labels to ProForma v2 notation
- Feature-generator configuration (e.g. {"ms2pip": {"model": "HCD"}, "deeplc": {}})

## Outputs

- PSM table annotated with predicted fragment ion intensity, retention time, and CCS features
- Feature table with target/decoy labels suitable for ristretto/Percolator semi-supervised rescoring

## How to apply

Run feature generators over the parsed PSM list: compute predicted MS2 fragment ion intensities with MS2PIP (selecting the model matching the acquisition, e.g. the HCD model), predicted retention time with DeepLC, and optionally CCS; the MS2Rescore default feature-generator configuration is {"basic": {}, "ms2pip": {"model": "HCD"}, "ms2": {}, "deeplc": {}}. Link each search-engine PSM back to its spectrum via configurable regex patterns (spectrum_id_pattern, psm_id_pattern, psm_id_rt_pattern, psm_id_im_pattern) so predictor outputs can be joined to the correct match. Parse heterogeneous search-engine modification labels into HUPO-PSI ProForma v2 notation using a configurable modification_mapping, and add fixed modifications not reported by search engines such as MaxQuant so rescoring features are computed on the correct peptidoform. Optionally include MHC binding-affinity predictions (e.g. NetMHCpan-style) — per the review only MHCquant and inSPIRE support affinity-based features among reviewed pipelines. Feed the annotated feature table to a semi-supervised learner such as ristretto (svm default, train_fdr 0.01) or Percolator.

## Related tools

- **MS2Rescore** (Orchestrates MS2PIP and DeepLC feature generation and modification parsing to ProForma for rescoring input) — https://github.com/compomics/ms2rescore
- **MS2PIP** (Predicts MS2 fragment ion intensities used as rescoring features (HCD model by default)) — https://github.com/compomics/ms2rescore
- **DeepLC** (Predicts retention time features from peptidoform sequences) — https://github.com/compomics/ms2rescore
- **NetMHCpan** (Predicted MHC binding affinity source usable as an affinity-based rescoring feature (supported by MHCquant and inSPIRE))
- **Prosit** (Deep-learning predictor of peptidoform properties (intensities, retention time, collisional cross section) referenced in the review)
- **Mumble** (PSM generator that expands peptidoforms with Unimod candidate modifications before feature annotation) — https://github.com/compomics/mumble

## Examples

```
ms2rescore --config-file xtandem-ms2rescore.toml identification.mzid
```

## Evaluation signals

- Feature table contains non-null intensity (MS2PIP), retention time (DeepLC), and basic features for every input PSM after spectrum linking via psm_id_pattern
- All peptidoforms are rendered in HUPO-PSI ProForma v2 notation with fixed modifications included (e.g. MaxQuant-reported fixed mods)
- Feature-generator configuration matches the acquisition mode, e.g. ms2pip model set to HCD for HCD data
- Downstream ristretto rescoring (train_fdr 0.01, svm) yields more peptide IDs at the same FDR threshold than the unrescored input, or similar IDs at a more stringent FDR
- Decoy PSMs are correctly identified via id_decoy_pattern so target/decoy labels in the feature table are consistent

## Limitations

- Search engines such as MaxQuant do not report fixed modifications that were part of the search; these must be added manually or features will be computed on incorrect peptidoforms
- Linking search engine PSMs to the original spectra via spectrum_id_pattern/psm_id_pattern is essential; misconfigured regex patterns silently break feature-to-PSM joins
- Only MHCquant and inSPIRE support predicted binding-affinity-based features among the reviewed pipelines; affinity features are not universally available
- Percolator is an external dependency with no version specified in the context, so compatibility must be verified

## Evidence
