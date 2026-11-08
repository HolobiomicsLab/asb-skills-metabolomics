---
name: compound-database-mass-search
description: Use when you have background-subtracted UPLC-HRMS peak lists (positive
  and/or negative ESI modes) of an herbal extract and need to assign putative compound
  identities from accurate masses before metabolite-bridging analysis.
when_to_use_negative:
- The goal is identifying in vivo metabolites rather than annotating the in vitro
  prototype list — use the MDRB bridging/scoring workflow instead, since MTSF-style
  matching alone 'only enables matching between single metabolite and the prototype
  database, and only utilizes the similarity of the secondary mass spectrometry maps'.
- The peak list still contains background/blank ions — database searching on unfiltered
  features will inflate the component inventory; apply PATBS background subtraction
  first.
- Structures are already known and the task is only metabolic-transformation mapping
  between prototypes and metabolites sharing a parent nucleus; database m/z search
  adds no bridging information.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: R
  role: R session used to load the UPLC-HRMS peak lists and run the PATBS/MDRB post-processing
    that produces the feature tables prior to database annotation
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Precise-and-thorough background-subtraction strips background/blank ions from
    the raw peak lists without requiring prior structural prediction, yielding the
    genuine component features to be searched
- name: MDRB
  role: Mass dataset relevance bridging post-processing constructs the component relationship
    network and bridges the annotated in vitro prototypes to in vivo metabolites via
    m/z, MS1/MS2 mass defect, and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
- name: PubChem
  role: Compound database queried with accurate m/z values to assign putative identities
    to prototype features
- name: ChemSpider
  role: Second compound database compared against PubChem for putative identity assignment
    of prototype components
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:009fbd445ac6b5fba38f514027285708d10438b6f2f5e460976966c78f97bf42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - ChemSpider
  - Chemspider
  - MDRB
  - PATBS
  - PubChem
  - Pubchem
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# compound-database-mass-search

## Summary

Annotate the prototype components of an in vitro UPLC-HRMS fingerprint (e.g. the 199-component CKXR list) by searching extracted m/z values against PubChem and ChemSpider, then rank hits by extracted peak intensity to identify dominant constituents such as amygdalin and prunasin.

## When to use

Use when you have background-subtracted UPLC-HRMS peak lists (positive and/or negative ESI modes) of an herbal extract and need to assign putative compound identities from accurate masses before metabolite-bridging analysis.

## When NOT to use

- The goal is identifying in vivo metabolites rather than annotating the in vitro prototype list — use the MDRB bridging/scoring workflow instead, since MTSF-style matching alone 'only enables matching between single metabolite and the prototype database, and only utilizes the similarity of the secondary mass spectrometry maps'.
- The peak list still contains background/blank ions — database searching on unfiltered features will inflate the component inventory; apply PATBS background subtraction first.
- Structures are already known and the task is only metabolic-transformation mapping between prototypes and metabolites sharing a parent nucleus; database m/z search adds no bridging information.

## Inputs

- UPLC-HRMS peak list of the in vitro herbal fingerprint in positive ESI mode
- UPLC-HRMS peak list of the in vitro herbal fingerprint in negative ESI mode
- Background/blank ion lists for PATBS subtraction

## Outputs

- Annotated prototype component table with putative identities per m/z and ionization-mode membership counts (e.g. 199 total; 96 positive / 136 negative / 33 both for CKXR)
- Abundance ranking of annotated prototypes by extracted peak intensity (e.g. amygdalin and prunasin at top)

## How to apply

First run background subtraction on the raw peak lists with the PATBS technique, which retains genuine sample components without requiring prior structural prediction, and tabulate the retained features by ionization mode (for CKXR: 96 in positive mode, 136 in negative mode, 33 in both, 199 total). Then take the m/z values of the retained prototype features and query the PubChem (https://pubchem.ncbi.nlm.nih.gov/) and ChemSpider (https://www.chemspider.com/) databases to assign putative identities to the prototype list. Finally, rank the annotated components by extracted peak intensity to surface the most abundant prototypes; for CKXR this step confirms amygdalin and prunasin as the highest-content constituents. The resulting annotated prototype table feeds downstream bridging steps (MDRB), where metabolite-prototype pairs are scored via structural similarity and minimal mass defect variations across m/z, MS1/MS2 mass defect, and MS2 fragment similarity.

## Related tools

- **R** (R session used to load the UPLC-HRMS peak lists and run the PATBS/MDRB post-processing that produces the feature tables prior to database annotation) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Precise-and-thorough background-subtraction strips background/blank ions from the raw peak lists without requiring prior structural prediction, yielding the genuine component features to be searched)
- **MDRB** (Mass dataset relevance bridging post-processing constructs the component relationship network and bridges the annotated in vitro prototypes to in vivo metabolites via m/z, MS1/MS2 mass defect, and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB
- **PubChem** (Compound database queried with accurate m/z values to assign putative identities to prototype features)
- **ChemSpider** (Second compound database compared against PubChem for putative identity assignment of prototype components)

## Examples

```
git clone https://github.com/933ZhangDD/MDRB  # R-based MDRB platform; run PATBS-filtered peak lists through MDRB, then annotate m/z against PubChem/ChemSpider
```

## Evaluation signals

- Mode-wise counts reconcile exactly: retained components in positive + negative modes with the both-mode overlap sum to the reported total (CKXR: 96 + 136 with 33 shared = 199).
- Abundance ranking places the reported dominant prototypes at the top (CKXR: amygdalin and prunasin highest relative contents); any deviation is flagged in the output summary.
- Every retained feature carries a putative annotation from PubChem and/or ChemSpider; features without a database hit are reported rather than silently dropped.
- Downstream MDRB scoring succeeds only when prototypes are correctly annotated, since metabolite-prototype pairing depends on structural similarity and minimal mass defect variations (m/z, MS1/MS2 mass defect, MS2 fragment similarity).

## Limitations

- Putative identities from PubChem/ChemSpider are based on mass matching and remain provisional; no statistical validation, replicate counts, or quantitative performance metrics are reported for the overall strategy.
- No specific mass spectrometry instrument model or data acquisition parameters are named, so mass tolerances for database search must be set from the local data.
- Identification of components (especially in vivo) remains difficult despite MS data post-processing; database search alone does not resolve structural isomers such as amygdalin vs prunasin.
- No software versions are stated for the MDRB platform or its data-processing steps.

## Evidence

- [intro] Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS): "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [intro] compared with Pubchem and Chemspider databases: "compared with Pubchem (https://pubchem.ncbi.nlm.nih.gov/) and Chemspider (https://www.chemspider.com/) databases"
- [abstract] The MDRB technology constructed a component relationship network for TCM: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency and facilitating the mapping of translational metabolic pathways."
- [abstract] We developed a scoring approach based on the structural similarity: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [intro] the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
