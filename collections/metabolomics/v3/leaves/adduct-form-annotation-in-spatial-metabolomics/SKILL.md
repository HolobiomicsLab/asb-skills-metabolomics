---
name: adduct-form-annotation-in-spatial-metabolomics
description: Use when input is imzML/ibd MS imaging data with MS1 scans only, and
  the goal is structural (level 2/3) metabolite or lipid annotations — including distinguishing
  adduct forms of the same molecule — rather than molecular-formula-only assignment.
when_to_use_negative:
- Input contains genuine MS/MS scans — use standard MS/MS library search or ms1-id's
  LC-MS/MS annotation (gnps.pkl) path instead of pseudo MS/MS generation
- Goal is molecular formula-level annotation only (MSI31 level 4) — a tool like METASPACE
  with target-decoy FDR may be more appropriate
- Data are LC-MS chromatographic profiles rather than imaging data — spatial correlation
  and 50-shared-pixel requirements do not apply; use ms1-id's lcms mode with chromatographic
  (XIC) correlation instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: ms1-id
  role: 'Core tool: preprocesses MS scans, builds pseudo MS/MS spectra from spatially
    correlated ion images, and performs precursor-tolerant reverse spectral matching
    against indexed GNPS libraries to annotate imzML imaging data'
  repo: https://github.com/Philipbear/ms1_id
- name: IDSL.CSA
  role: Prior proof-of-principle for matching pseudo MS/MS spectra, which ms1-id extends
    to MS1-only imaging data without extra in-source fragmentation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:9b22afccc30ab8cfe58bd01b21a63e81a73fca98dfa6454db5493f7e25d81b68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - IDSL.CSA
  - ms1-id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Adduct-form annotation in spatial metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates MS1-only mass spectrometry imaging data at structurally resolved adduct level (e.g. PC(16:0/18:1) as [M+H]+ and [M+Na]+) using ms1-id, which builds pseudo MS/MS spectra from spatially correlated in-source fragment ion images and matches them against indexed reference MS/MS libraries.

## When to use

Use when input is imzML/ibd MS imaging data with MS1 scans only, and the goal is structural (level 2/3) metabolite or lipid annotations — including distinguishing adduct forms of the same molecule — rather than molecular-formula-only assignment.

## When NOT to use

- Input contains genuine MS/MS scans — use standard MS/MS library search or ms1-id's LC-MS/MS annotation (gnps.pkl) path instead of pseudo MS/MS generation
- Goal is molecular formula-level annotation only (MSI31 level 4) — a tool like METASPACE with target-decoy FDR may be more appropriate
- Data are LC-MS chromatographic profiles rather than imaging data — spatial correlation and 50-shared-pixel requirements do not apply; use ms1-id's lcms mode with chromatographic (XIC) correlation instead

## Inputs

- imzML/ibd mass spectrometry imaging dataset (e.g. mouse brain MALDI-FTICR from MetaboLights MTBLS313)
- Indexed GNPS MS/MS spectral library for imaging data (gnps_minmz100.pkl and gnps_minmz100_k10.pkl, built with peak_scale_k=10 and peak_intensity_power=0.5)

## Outputs

- Annotation table (ms1_id_annotations_derep.tsv, e.g. mouse_brain_msi_annotations.csv) with metabolite/adduct annotations and reverse cosine scores
- Per-annotation ion images showing spatial distribution of the annotated ion forms (e.g. PC(16:0/18:1) [M+H]+ score 0.85 and [M+Na]+ score 0.77, hemin cation)

## How to apply

Run ms1-id on imzML/ibd files, preprocessing each MS scan with moving-average noise reduction (100 Da window, baseline = 5x mean of lowest 5% ions, sn_factor 3.0). Compile pseudo MS/MS spectra by extracting ion images at 10 ppm tolerance and spatially correlating ion image pairs (≥50 shared non-zero pixels, correlation cutoff 0.85). Index the GNPS MS/MS library with mass-dependent peak intensity scaling (k = 10) and square-root transformation (power 0.5) so low-energy in-source fragments can match higher-collision-energy reference spectra; use the gnps_minmz100 library variant since fragments with m/z < 100 are not captured in imaging data. Search with precursor-tolerant reverse spectral matching using MS imaging cutoffs: minimum reverse cosine score 0.7, minimum 3 matched peaks, minimum spectral usage 0.05 — the reverse search accommodates multiple ion forms (adducts, multimers) of one precursor simultaneously. Verify recovered annotations by checking that the same metabolite is annotated under distinct adduct forms with distinct reverse cosine scores and that cation ion images are spatially coherent with their in-source fragments.

## Related tools

- **ms1-id** (Core tool: preprocesses MS scans, builds pseudo MS/MS spectra from spatially correlated ion images, and performs precursor-tolerant reverse spectral matching against indexed GNPS libraries to annotate imzML imaging data) — https://github.com/Philipbear/ms1_id
- **IDSL.CSA** (Prior proof-of-principle for matching pseudo MS/MS spectra, which ms1-id extends to MS1-only imaging data without extra in-source fragmentation)

## Examples

```
ms1_id msi --input_dir msi --libs db/gnps_minmz100.pkl db/gnps_minmz100_k10.pkl --n_cores 12
```

## Evaluation signals

- PC(16:0/18:1) must be annotated in both [M+H]+ and [M+Na]+ forms with reverse cosine scores of 0.85 and 0.77 respectively
- All annotations must meet MS imaging cutoffs: reverse cosine ≥ 0.7, ≥ 3 matched peaks, spectral usage ≥ 0.05
- Pseudo MS/MS pairs used in correlation must share ≥ 50 non-zero pixels with spatial correlation ≥ 0.85 and 10 ppm ion-image extraction tolerance
- The hemin cation ion image must show spatial patterns highly similar to its in-source fragments
- Only reference library peaks with m/z ≥ 100 should be matched (gnps_minmz100 variant), since imaging MS does not detect lower fragments

## Limitations

- Reverse cosine scores for adduct forms can be moderate (e.g. [M+Na]+ scored 0.77 vs 0.85 for [M+H]+), reflecting mismatch between low-energy in-source fragments and higher-collision-energy reference spectra
- Fragments with m/z < 100 cannot be annotated in imaging data and are removed from the imaging library variant
- Annotation confidence is level 2/3 and depends on spatial correlation of in-source fragments, so metabolites lacking correlated fragments will not be recovered

## Evidence

- [other] ms1-id annotated PC(16:0/18:1) in mouse brain MS imaging data in both [M+H]+ and [M+Na]+ forms, achieving reverse cosine scores of 0.85 and 0.77, respectively.: [evidence span withheld — anchor status: not_found]
- [discussion] For MS imaging data, we used: minimum score, 0.7; minimum matched peaks, 3; minimum spectral usage, 0.05: "For MS imaging data, we used: minimum score, 0.7; minimum matched peaks, 3; minimum spectral usage, 0.05"
- [discussion] A minimum of 50 shared pixels with non-zero intensities between two ion images is required: "A minimum of 50 shared pixels with non-zero intensities between two ion images is required"
- [readme] For MS imaging data (fragments with mz < 100 are removed, as they are not usually included in MS imaging data): [evidence span withheld — anchor status: external_doc]
- [abstract] precursor-tolerant reverse spectral matching that accommodates multiple ion forms simultaneously: "precursor-tolerant reverse spectral matching that accommodates multiple ion forms simultaneously"
