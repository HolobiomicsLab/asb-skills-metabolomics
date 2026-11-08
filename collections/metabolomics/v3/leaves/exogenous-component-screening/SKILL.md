---
name: exogenous-component-screening
description: Use when you have paired pre-dosing and post-dosing LC-MS/MS feature
  lists for multiple biological matrices (e.g., plasma, lung, urine, feces) and need
  to isolate drug/herb-derived exogenous peaks and count prototype vs metabolite components.
when_to_use_negative:
- The input is already a filtered exogenous-component feature table (background subtraction
  has already been done against blanks)
- The goal is targeted quantification of known analytes rather than non-targeted discovery
  of dosage-related components
- No matched pre-dosing blank matrix is available for the post-dosing sample, since
  PATBS background subtraction requires sample/blank comparison
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PATBS
  role: Non-targeted background-subtraction technique that compares post-dosing features
    against pre-dosing blank matrices to identify exogenous metabolites without requiring
    prior structural prediction
- name: R
  role: R-based environment in which the PATBS processing and MDRB data post-processing
    are run
  repo: https://github.com/933ZhangDD/MDRB
- name: MDRB
  role: Downstream data post-processing that bridges retained exogenous metabolites
    to prototypes via m/z, MS1/MS2 mass defect, and MS2 fragment similarity, constructing
    a component relationship network
  repo: https://github.com/933ZhangDD/MDRB
- name: Metworks 1.3.0.200
  role: Provides the known predictable metabolic responses used as the basis for metabolite-prototype
    bridging
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:7719956fe70b6d343e1bb7f37ce342063bf83d12db93f1cfee21141e114f8fb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# exogenous-component-screening

## Summary

Screens LC-MS/MS features from dosed biological matrices against pre-dosing (blank) matrix backgrounds using the PATBS non-targeted background-subtraction technique to retain only exogenous, dosage-related components, then classifies them as prototypes versus metabolites and tallies occurrences per matrix.

## When to use

Use when you have paired pre-dosing and post-dosing LC-MS/MS feature lists for multiple biological matrices (e.g., plasma, lung, urine, feces) and need to isolate drug/herb-derived exogenous peaks and count prototype vs metabolite components.

## When NOT to use

- The input is already a filtered exogenous-component feature table (background subtraction has already been done against blanks)
- The goal is targeted quantification of known analytes rather than non-targeted discovery of dosage-related components
- No matched pre-dosing blank matrix is available for the post-dosing sample, since PATBS background subtraction requires sample/blank comparison

## Inputs

- Pre-dosing (blank) LC-MS/MS feature lists for each biological matrix (plasma, lung, urine, feces)
- Post-dosing LC-MS/MS feature lists for each biological matrix
- Prototype component list for the herb/compound of interest (e.g., CKXR prototypes)

## Outputs

- Annotated exogenous-component table (e.g., patbs_exogenous_components.csv) with prototype/metabolite labels per component
- Per-matrix component counts (e.g., plasma, lung, urine, feces tallies)
- Total exogenous-component inventory split into prototypes vs metabolites

## How to apply

Load the pre-dosing and post-dosing LC-MS/MS data files for each biological matrix (plasma, lung, urine, feces) into the R-based PATBS processing environment. Run PATBS background subtraction by comparing post-dosing sample feature lists against the corresponding pre-dosing (blank) matrices, removing endogenous/background components and retaining exogenous, dosage-related peaks. Annotate the retained exogenous features against the prototype component list (e.g., CKXR for semen Armeniacae Amarum), labeling prototypes separately from metabolites. Tabulate component occurrences per matrix and export an annotated table (e.g., patbs_exogenous_components.csv) with prototype/metabolite labels and per-matrix assignments. In the reference mouse study this yielded 351 CKXR-related exogenous components (41 prototypes, 310 metabolites) distributed as 87 plasma, 53 lung, 342 urine, and 42 feces; downstream bridging to prototypes can be refined with MDRB, which integrates m/z, MS1/MS2 mass defect, and MS2 fragment similarity.

## Related tools

- **PATBS** (Non-targeted background-subtraction technique that compares post-dosing features against pre-dosing blank matrices to identify exogenous metabolites without requiring prior structural prediction)
- **R** (R-based environment in which the PATBS processing and MDRB data post-processing are run) — https://github.com/933ZhangDD/MDRB
- **MDRB** (Downstream data post-processing that bridges retained exogenous metabolites to prototypes via m/z, MS1/MS2 mass defect, and MS2 fragment similarity, constructing a component relationship network) — https://github.com/933ZhangDD/MDRB
- **Metworks 1.3.0.200** (Provides the known predictable metabolic responses used as the basis for metabolite-prototype bridging)

## Evaluation signals

- Output table contains exactly the expected number of exogenous components (e.g., 351 = 41 prototypes + 310 metabolites in the reference CKXR mouse study)
- Per-matrix counts match the reported PATBS results (87 plasma, 53 lung, 342 urine, 42 feces)
- Every retained feature in the output is absent from (or reduced in) the corresponding pre-dosing blank matrix, confirming exogenous origin
- Each component carries an explicit prototype or metabolite label, and prototype/metabolite tallies sum to the total component count
- MDRB-bridged metabolite assignments are consistent across the three integrated dimensions: m/z, MS1/MS2 mass defect, and MS2 fragment similarity

## Limitations

- Identification of in vivo components remains difficult despite MS data post-processing techniques
- The precursor MTSF-style matching only matches single metabolites to a prototype database using MS2 map similarity; PATBS/MDRB were developed to overcome this, but bridging still depends on mass-defect windows and fragment-similarity scoring
- No specific mass spectrometry instrument model or data acquisition parameters are stated, and no software versions are reported for the MDRB/data-processing platform, complicating exact reproduction

## Evidence

- [abstract] PATBS non-targeted identification: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [abstract] MDRB three-dimension scoring: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [abstract] combined strategy in mice: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy"
- [intro] mass defect bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
