---
name: patbs-background-subtraction
description: Use when raw UPLC-HRMS peak lists in positive and negative ESI modes
  contain background/blank ions that must be removed to enumerate genuine in vitro
  sample components (prototypes) before annotating identities and ranking abundances.
when_to_use_negative:
- Input is already a cleaned, background-subtracted feature table — applying PATBS
  again would strip genuine sample components.
- The goal is characterizing in vivo metabolites and bridging them to prototypes via
  mass defect and MS2 fragment matching — that is the role of the MDRB scoring approach,
  not PATBS.
- The task is matching single metabolites against a prototype database using only
  secondary mass-spectrometry map similarity — the article notes this MTSF-style approach
  is narrower than the PATBS+MDRB strategy.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Environment in which the PATBS background subtraction and mass-dataset post-processing
    (MDRB) analysis is executed
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Self-developed precise-and-thorough background-subtraction technique that
    performs non-targeted identification of sample components from UPLC-HRMS data
- name: MDRB
  role: Downstream data post-processing that bridges the PATBS-derived prototype list
    to in vivo metabolites via m/z, MS1/MS2 mass defect and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
- name: Pubchem
  role: External database used to annotate retained prototype components by m/z
- name: Chemspider
  role: External database used to annotate retained prototype components by m/z
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:1f39669482df2faeb73dee16fef0548ddfc79dbf30507e5c34faa92a7d133c06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - Chemspider
  - MDRB
  - PATBS
  - Pubchem
  - R
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# patbs-background-subtraction

## Summary

A non-targeted, R-based background-subtraction workflow (PATBS, precise-and-thorough background-subtraction) that strips background/blank ions from UPLC-HRMS peak lists in positive and negative ESI modes to build a genuine in vitro component (prototype) list for a TCM herb without requiring prior prediction of component structures.

## When to use

Use when raw UPLC-HRMS peak lists in positive and negative ESI modes contain background/blank ions that must be removed to enumerate genuine in vitro sample components (prototypes) before annotating identities and ranking abundances.

## When NOT to use

- Input is already a cleaned, background-subtracted feature table — applying PATBS again would strip genuine sample components.
- The goal is characterizing in vivo metabolites and bridging them to prototypes via mass defect and MS2 fragment matching — that is the role of the MDRB scoring approach, not PATBS.
- The task is matching single metabolites against a prototype database using only secondary mass-spectrometry map similarity — the article notes this MTSF-style approach is narrower than the PATBS+MDRB strategy.

## Inputs

- Raw UPLC-HRMS fingerprint peak lists in positive ESI mode
- Raw UPLC-HRMS fingerprint peak lists in negative ESI mode
- Background/blank run peak lists used by PATBS for subtraction
- PubChem and ChemSpider databases for component annotation

## Outputs

- In vitro prototype component table with mode-wise counts (positive / negative / both modes)
- Ranked component abundance list (e.g. amygdalin and prunasin as top-content prototypes for CKXR)
- Prototype list serving as input to MDRB prototype–metabolite bridging

## How to apply

Load the raw CKXR (semen Armeniacae Amarum) UPLC-HRMS fingerprint peak lists from both positive and negative ESI modes into R, then apply PATBS to strip background/blank ions and retain only genuine sample components — PATBS is non-targeted, so no prediction of herbal component structures is needed beforehand. Tabulate the retained features by ionization mode, counting components detected in positive mode, negative mode, and both modes, and reconcile the split against the expected 96 / 136 / 33 distribution summing to 199 total prototypes. Annotate the retained component m/z values against the PubChem and ChemSpider databases to assign putative identities, then rank annotated components by extracted peak intensity to identify the highest-content prototypes (amygdalin and prunasin for CKXR). The resulting prototype list is the input for downstream in vivo metabolite bridging via the MDRB technique, which constructs a component relationship network between prototypes and metabolites using m/z, MS1/MS2 mass defect, and MS2 fragment similarity.

## Related tools

- **R** (Environment in which the PATBS background subtraction and mass-dataset post-processing (MDRB) analysis is executed) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Self-developed precise-and-thorough background-subtraction technique that performs non-targeted identification of sample components from UPLC-HRMS data)
- **MDRB** (Downstream data post-processing that bridges the PATBS-derived prototype list to in vivo metabolites via m/z, MS1/MS2 mass defect and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB
- **Pubchem** (External database used to annotate retained prototype components by m/z)
- **Chemspider** (External database used to annotate retained prototype components by m/z)

## Examples

```
git clone https://github.com/933ZhangDD/MDRB
```

## Evaluation signals

- The rebuilt prototype table must reproduce the reported mode-wise counts: 199 total components, 96 in positive ion mode, 136 in negative ion mode, 33 in both modes.
- The intensity ranking must list amygdalin and prunasin as the highest-content in vitro prototypes; discrepancies are flagged in the output summary.
- Retained features must be genuine sample components — background/blank ions present in control runs should be removed by the subtraction step.
- Annotated prototype identities should resolve against PubChem/ChemSpider database entries within the m/z matching used in the workflow.
- The resulting prototype list must be usable as input to MDRB bridging (m/z, MS1/MS2 mass defect, MS2 fragment similarity) to construct the component relationship network.

## Limitations

- No software version is stated for the PATBS technique or the data-processing platform, and no changelog is available for the MDRB source repository.
- No specific mass spectrometry instrument model or data acquisition parameters are named in the article, hindering exact reproduction of acquisition conditions.
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the PATBS + MDRB strategy.
- The article acknowledges that identification of components remains difficult even with mass-spectrometry data post-processing techniques.

## Evidence

- [intro] Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components.: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [intro] First, we realized the non-target identification of metabolites by our self-developed PATBS technique.: "First, we realized the non-target identification of metabolites by our self-developed PATBS technique."
- [abstract] the developed MDRB platform is publicly available for ( https://github.com/933ZhangDD/MDRB ): "the developed MDRB platform is publicly available for ( https://github.com/933ZhangDD/MDRB )"
