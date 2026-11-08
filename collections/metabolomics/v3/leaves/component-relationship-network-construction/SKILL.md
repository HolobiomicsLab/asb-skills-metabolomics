---
name: component-relationship-network-construction
description: Use when in vivo metabolite feature lists (m/z, MS1/MS2 mass defects,
  MS2 fragment spectra) must be linked to in vitro prototype components from a non-targeted
  identification step to map metabolic transformations.
when_to_use_negative:
- You only need single-metabolite-to-prototype-database matching using MS2 map similarity
  alone — the article notes MTSF handles that narrower task, so MDRB's three-dimensional
  scoring is unnecessary.
- No in vitro prototype component list exists (e.g., no PATBS non-targeted identification
  output), so there is nothing to bridge metabolites to.
- Metabolites and candidate prototypes share no common parent nucleus structure, so
  mass-defect-based bridging assumptions do not hold.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing pipeline that sequentially
    analyzes m/z, MS1/MS2 mass defect, and MS2 fragment similarity and builds the
    component relationship network
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Produces the non-targeted in vitro prototype (component) list that metabolites
    are bridged back to
- name: Metworks 1.3.0.200
  role: Provides the known predictable metabolic responses used to cover prototype-metabolite
    links along predictable pathways
- name: R
  role: Language/environment in which the MDRB post-processing technique is implemented
    and run
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:d161d85588fbd81f9379a4ad7957148bcd8ec0605f5c91eafd37cf765bd33462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# component-relationship-network-construction

## Summary

Bridges in vivo TCM metabolites to their in vitro prototype components by scoring metabolite-prototype pairs on structural similarity and minimal mass defect variations, then assembles the results into a component relationship network (demonstrated with MDRB on Semen Armeniacae Amarum/CKXR in mice).

## When to use

Use when in vivo metabolite feature lists (m/z, MS1/MS2 mass defects, MS2 fragment spectra) must be linked to in vitro prototype components from a non-targeted identification step to map metabolic transformations.

## When NOT to use

- You only need single-metabolite-to-prototype-database matching using MS2 map similarity alone — the article notes MTSF handles that narrower task, so MDRB's three-dimensional scoring is unnecessary.
- No in vitro prototype component list exists (e.g., no PATBS non-targeted identification output), so there is nothing to bridge metabolites to.
- Metabolites and candidate prototypes share no common parent nucleus structure, so mass-defect-based bridging assumptions do not hold.

## Inputs

- In vivo metabolite feature list with m/z, MS1 and MS2 mass defects, and MS2 fragment spectra (e.g., from plasma, urine, feces, lung matrices)
- In vitro prototype (parent-component) list produced by PATBS non-targeted identification
- Predictable metabolic transformation rules (e.g., via Metworks 1.3.0.200)

## Outputs

- Component relationship network (edges bridging in vivo metabolites to in vitro prototypes)
- Per-matrix counts of characterized metabolites (e.g., 66 plasma / 222 urine / 30 feces / 40 lung for CKXR)
- Characterization rate versus manual baseline (227 metabolites at 65% vs 110 at 31%)
- Translational metabolic pathway mappings (metabolic transformation pathways of CKXR in mice)

## How to apply

Load the CKXR metabolite feature list (m/z, MS1/MS2 mass defects, MS2 fragment spectra) and the prototype (in vitro component) list produced by the PATBS non-targeted identification step. Run the MDRB R-based data post-processing pipeline (https://github.com/933ZhangDD/MDRB), which sequentially analyzes three dimensions of mass spectral data: m/z, mass defect of MS1 and MS2, and similarity of MS2 fragments — the premise being that the mass defect of MS2 fragments of prototypes and their metabolites falls within a certain range that bridges the two. Score metabolite-prototype pairs on structural similarity and minimal mass defect variations; predictable metabolic responses can be based on Metworks 1.3.0.200 to cover prototype-metabolite links beyond unpredictable pathways. Construct the component relationship network linking in vivo metabolites to in vitro prototypes across the plasma, urine, feces, and lung matrices, and count characterized metabolites per matrix (the CKXR benchmark: 66 plasma, 222 urine, 30 feces, 40 lung; 227 total, 65%).

## Related tools

- **MDRB** (R-based mass dataset relevance bridging post-processing pipeline that sequentially analyzes m/z, MS1/MS2 mass defect, and MS2 fragment similarity and builds the component relationship network) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Produces the non-targeted in vitro prototype (component) list that metabolites are bridged back to)
- **Metworks 1.3.0.200** (Provides the known predictable metabolic responses used to cover prototype-metabolite links along predictable pathways)
- **R** (Language/environment in which the MDRB post-processing technique is implemented and run) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Per-matrix characterized metabolite counts match the reported benchmark: 66 plasma, 222 urine, 30 feces, 40 lung.
- Total characterized metabolites reach 227 (65%), reproducibly exceeding the manual baseline of 110 metabolites (31%).
- Every network edge connects an in vivo metabolite to an in vitro prototype with a recorded structural-similarity and mass-defect-variation score.
- Bridged metabolite-prototype pairs show MS2 fragment mass defects within the expected range, consistent with a shared parent nucleus structure.
- The three dimensions (m/z, MS1/MS2 mass defect, MS2 fragment similarity) are all exercised in the sequential analysis, not just MS2 map similarity.

## Limitations

- Despite MS data post-processing techniques, identification of in vivo components remains difficult.
- No software versions are stated for the MDRB technique or the data-processing platform, and the repository has no changelog.
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy.
- No specific mass spectrometry instrument model or data acquisition parameters are named, complicating exact reproduction.

## Evidence

- [abstract] MDRB network construction claim: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency"
- [abstract] Scoring approach: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [abstract] Three data dimensions: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] MS2 mass-defect bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging"
- [intro] MTSF narrower-scope contrast: "However, this technique only enables matching between single metabolite and the prototype database, and only utilizes the similarity of the secondary mass spectrometry maps."
