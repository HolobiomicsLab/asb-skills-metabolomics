---
name: tcm-metabolite-characterization
description: Use when you have LC-MS feature lists from in vivo TCM dosing (plasma,
  urine, feces, tissue) plus an in vitro prototype component list and need to characterize
  metabolites and bridge them to parent prototypes.
when_to_use_negative:
- The analysis only requires matching a single metabolite against a prototype database
  using MS2 map similarity alone (e.g., MTSF-style matching) rather than integrated
  m/z + mass defect + fragment scoring
- The in vivo feature list has no corresponding in vitro prototype/component data
  to bridge against, so prototype matching and metabolic-pathway matching cannot be
  performed
- You need statistical validation or quantitative performance metrics of the identification
  strategy — the paper reports none and acknowledges in vivo component identification
  remains difficult
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MDRB
  role: Core R-based mass dataset relevance bridging pipeline that sequentially analyzes
    m/z, MS1/MS2 mass defects, and MS2 fragment similarity to score metabolite-prototype
    pairs and build the component relationship network
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Supplies the non-targeted in vitro prototype (exposed component) list that
    in vivo metabolites are bridged against
- name: Metworks
  role: Version 1.3.0.200 provides the known predictable metabolic responses used
    in the predictable metabolic pathway matching step
- name: R
  role: Implementation language of the MDRB data post-processing pipeline
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:06be134f28e7970f594b5b9a875278131ceeae30e36ab4038a67812ce92d2a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# tcm-metabolite-characterization

## Summary

Characterize traditional Chinese medicine (TCM) metabolites in vivo by bridging them to in vitro prototypes using the MDRB R-based data post-processing technique, which sequentially scores m/z, MS1/MS2 mass defects, and MS2 fragment similarity. It raises characterization rates substantially over manual annotation (e.g., 227 metabolites at 65% vs 110 at 31% for semen Armeniacae Amarum/CKXR in mice).

## When to use

Use when you have LC-MS feature lists from in vivo TCM dosing (plasma, urine, feces, tissue) plus an in vitro prototype component list and need to characterize metabolites and bridge them to parent prototypes.

## When NOT to use

- The analysis only requires matching a single metabolite against a prototype database using MS2 map similarity alone (e.g., MTSF-style matching) rather than integrated m/z + mass defect + fragment scoring
- The in vivo feature list has no corresponding in vitro prototype/component data to bridge against, so prototype matching and metabolic-pathway matching cannot be performed
- You need statistical validation or quantitative performance metrics of the identification strategy — the paper reports none and acknowledges in vivo component identification remains difficult

## Inputs

- In vivo metabolite feature list with m/z, MS1/MS2 mass defects, and MS2 fragment spectra from dosed matrices (plasma, urine, feces, lung/tissue)
- In vitro prototype (parent component) list produced by PATBS non-targeted identification
- Prototype structure database for structural-similarity scoring (article compared candidates against PubChem and ChemSpider)

## Outputs

- Characterized metabolite list with bridged prototype assignments (227 metabolites, 65% characterization rate for CKXR)
- Per-matrix characterized metabolite counts (66 plasma, 222 urine, 30 feces, 40 lung)
- Component relationship network connecting in vivo metabolites to in vitro prototypes
- Comparison table against manual characterization baseline (110 metabolites, 31%)

## How to apply

First obtain the in vitro prototype list from a non-targeted PATBS (precise-and-thorough background-subtraction) identification step, and the in vivo metabolite feature list containing m/z, MS1/MS2 mass defects, and MS2 fragment spectra. Run the MDRB R pipeline (https://github.com/933ZhangDD/MDRB), which sequentially analyzes the three dimensions of mass spectral data — m/z, mass defect of MS1 and MS2, and similarity of MS2 fragments. Score metabolite-prototype pairs based on structural similarity and minimal mass defect variations; optionally incorporate predictable metabolic responses (the paper used Metworks 1.3.0.200) for predictable pathway matching. Construct the component relationship network linking in vivo metabolites to in vitro prototypes across all matrices (for CKXR: 66 plasma, 222 urine, 30 feces, 40 lung), then count characterized metabolites and compute the characterization rate against a manual baseline. Validation is by reproducing the reported counts and per-matrix distribution and recording the comparison against the manual characterization.

## Related tools

- **MDRB** (Core R-based mass dataset relevance bridging pipeline that sequentially analyzes m/z, MS1/MS2 mass defects, and MS2 fragment similarity to score metabolite-prototype pairs and build the component relationship network) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Supplies the non-targeted in vitro prototype (exposed component) list that in vivo metabolites are bridged against)
- **Metworks** (Version 1.3.0.200 provides the known predictable metabolic responses used in the predictable metabolic pathway matching step)
- **R** (Implementation language of the MDRB data post-processing pipeline) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Total characterized metabolite count and characterization rate match the reported 227 (65%) versus the manual baseline of 110 (31%)
- Per-matrix counts match the reported distribution: 66 plasma, 222 urine, 30 feces, 40 lung
- Every characterized metabolite is bridged to a prototype via the component relationship network with a score based on structural similarity and minimal mass defect variations
- The three mass-spectral dimensions (m/z, MS1/MS2 mass defect, MS2 fragment similarity) were analyzed sequentially rather than only MS2 map similarity (the MTSF limitation)
- Predictable metabolic pathways were checked against the Metworks 1.3.0.200 response set in addition to unpredictable pathway matching

## Limitations

- No software versions stated for the MDRB technique or its data-processing platform, and no changelog is available for the MDRB repository
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy
- Identification of in vivo components remains difficult despite MS data post-processing
- No specific mass spectrometry instrument model or data acquisition parameters are named, limiting reproducibility of acquisition settings
- The scoring approach depends on prototypes sharing the same parent nucleus structure; components without in vitro prototypes cannot be bridged

## Evidence

- [abstract] Three-dimension integration rationale: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [abstract] Scoring approach definition: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes"
- [intro] MTSF limitation motivating MDRB: "However, this technique only enables matching between single metabolite and the prototype database, and only utilizes the similarity of the secondary mass spectrometry maps."
- [abstract] Component relationship network output: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency"
- [intro] Metworks version used for predictable pathways: "We based the known predictable metabolic responses provided by Metworks 1.3.0.200."
- [abstract] MDRB public availability: "the developed MDRB platform is publicly available for ( https://github.com/933ZhangDD/MDRB )"
