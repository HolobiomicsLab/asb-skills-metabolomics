---
name: prototype-metabolite-annotation
description: Use when non-targeted LC-MS/MS feature lists from dosed vs. blank biological
  matrices (e.g., mouse plasma, lung, urine, feces) must be annotated as prototypes
  or metabolites of a known herbal prototype component set.
when_to_use_negative:
- No prototype component database exists for the herb under study, since MDRB bridging
  requires prototypes as anchors for mass-defect and fragment-similarity scoring
- The goal is targeted quantification of known compounds rather than non-targeted
  annotation of exogenous components in dosed vs. blank samples
- Only MS2 map similarity matching to a single prototype is needed (MTSF-style), which
  the article notes cannot exploit mass-defect dimensions or build relationship networks
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PATBS
  role: Self-developed non-targeted background-subtraction technique used to identify
    exogenous metabolites from dosed vs. blank LC-MS/MS data
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing that scores metabolite-prototype
    pairs via m/z, MS1/MS2 mass defect, and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
- name: Metworks 1.3.0.200
  role: Provides the known predictable metabolic responses (biotransformations) applied
    to prototype structures during metabolite prediction
- name: R
  role: Environment in which the PATBS/MDRB processing workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:e5c5fb418366036e486b40b8328f67e1630cc211c18efe7533408fd19f0ad527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# prototype-metabolite-annotation

## Summary

Bridges exogenous LC-MS/MS features to herbal prototype components and their metabolites using PATBS background subtraction plus MDRB scoring over m/z, MS1/MS2 mass defect, and MS2 fragment similarity, as demonstrated for semen Armeniacae Amarum (CKXR) in mice.

## When to use

Use when non-targeted LC-MS/MS feature lists from dosed vs. blank biological matrices (e.g., mouse plasma, lung, urine, feces) must be annotated as prototypes or metabolites of a known herbal prototype component set.

## When NOT to use

- No prototype component database exists for the herb under study, since MDRB bridging requires prototypes as anchors for mass-defect and fragment-similarity scoring
- The goal is targeted quantification of known compounds rather than non-targeted annotation of exogenous components in dosed vs. blank samples
- Only MS2 map similarity matching to a single prototype is needed (MTSF-style), which the article notes cannot exploit mass-defect dimensions or build relationship networks

## Inputs

- Pre-dosing (blank) and post-dosing LC-MS/MS data files per biological matrix (plasma, lung, urine, feces) from dosed mice
- PATBS-derived exogenous feature lists (post-dosing minus blank background subtraction)
- Herbal prototype component database for the studied TCM (e.g., CKXR prototype list)
- Predictable metabolic responses from Metworks 1.3.0.200

## Outputs

- Annotated exogenous component table with prototype/metabolite labels and per-matrix assignments (patbs_exogenous_components.csv)
- Component relationship network bridging in vitro prototypes to in vivo metabolites
- Per-matrix component counts (e.g., 87 plasma, 53 lung, 342 urine, 42 feces; 351 total = 41 prototypes + 310 metabolites)

## How to apply

Load pre-dosing and post-dosing LC-MS/MS data per biological matrix (plasma, lung, urine, feces) and run PATBS background subtraction, comparing post-dosing feature lists against the corresponding pre-dosing blank matrices to retain dosage-related exogenous peaks. Annotate retained features against the CKXR prototype component list, labeling each as prototype or metabolite. Bridge metabolites to prototypes with the MDRB R-based post-processing, which sequentially analyzes three dimensions of mass spectral data — m/z, mass defect of MS1 and MS2, and similarity of MS2 fragments — via a scoring approach based on structural similarity and minimal mass defect variation between metabolites and prototypes; Metworks 1.3.0.200 supplies the known predictable metabolic responses (biotransformations) applied to prototype structures. Tabulate component occurrences per matrix (expected for CKXR: 87 plasma, 53 lung, 342 urine, 42 feces) and write the annotated table with prototype/metabolite labels. Optionally construct the component relationship network that correlates in vitro and in vivo data for components sharing the same parent nucleus.

## Related tools

- **PATBS** (Self-developed non-targeted background-subtraction technique used to identify exogenous metabolites from dosed vs. blank LC-MS/MS data)
- **MDRB** (R-based mass dataset relevance bridging post-processing that scores metabolite-prototype pairs via m/z, MS1/MS2 mass defect, and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB
- **Metworks 1.3.0.200** (Provides the known predictable metabolic responses (biotransformations) applied to prototype structures during metabolite prediction)
- **R** (Environment in which the PATBS/MDRB processing workflow is executed)

## Evaluation signals

- Output table contains exactly 351 exogenous components (41 prototypes + 310 metabolites) for the CKXR mouse study
- Per-matrix counts match the reported PATBS results: 87 plasma, 53 lung, 342 urine, 42 feces
- Every metabolite call is bridged to a prototype with consistent m/z, MS1/MS2 mass defect, and MS2 fragment similarity scores
- Annotated table (patbs_exogenous_components.csv) carries explicit prototype/metabolite labels and per-matrix assignments
- Component relationship network connects in vitro prototypes and in vivo metabolites sharing the same parent nucleus structure

## Limitations

- No software versions are stated for the MDRB technique or the data-processing platform beyond Metworks 1.3.0.200
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy
- No specific mass spectrometry instrument model or data acquisition parameters are named
- Identification of in vivo components remains difficult despite MS data post-processing techniques

## Evidence

- [abstract] scoring approach for metabolite-prototype pairs: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [abstract] three dimensions of mass spectral data: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] MDRB bridges prototype and metabolites: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [intro] mass-defect basis for bridging: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [intro] Metworks as metabolism knowledge base: "We based the known predictable metabolic responses provided by Metworks 1.3.0.200."
