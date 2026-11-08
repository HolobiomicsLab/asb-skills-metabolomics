---
name: ms2-similarity-network-tool-comparison
description: Use when a feature table carries network-propagated level 3 annotations
  (e.g. MetDNA3 output on Orbitrap data) and you need to quantify how many annotations
  independent in-silico tools support.
when_to_use_negative:
- Annotations are already level 1/2 confirmed with chemical standards — concordance
  testing adds no orthogonal information and cross-tool agreement would be circular
- Input features come from non-Orbitrap instruments not represented in the 20-dataset
  benchmark, so the 4,302-feature denominator and reported fractions do not apply
- You need de novo annotation rather than validation of existing network-based level
  3 annotations
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetDNA3
  role: Source of the network-based level 3 annotations being corroborated; implemented
    in MrnAnnoAlgo3 and the MetDNA3 webserver
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: CFM-ID
  role: Independent in-silico MS/MS tool used for cross evaluation of KGMN/level 3
    annotations
- name: MetFrag
  role: Independent candidate-ranking annotation tool used for cross evaluation of
    KGMN/level 3 annotations
- name: MS-FINDER
  role: Independent formula/MS2 annotation tool used for cross evaluation of KGMN/level
    3 annotations
- name: SIRIUS
  role: Independent isotope-pattern/MS2 annotation tool used for cross evaluation
    of KGMN/level 3 annotations
- name: MetDNA2
  role: Predecessor package whose README documents cross-evaluation tutorials for
    MetFrag, CFM-ID, and MS-FINDER used in this comparison workflow
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:dcbe87a3e817ffc79ee36562caec5c108e2f23c282455f22b152cf47e38f04d8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - CFM-ID
  - FBMN
  - GNPS
  - IIMN
  - MS-FINDER
  - MetDNA2
  - MetDNA3
  - MetFrag
  - NetID
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ms2-similarity-network-tool-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-validates network-based (MetDNA3) level 3 metabolite annotations against four independent, non-network annotation tools (CFM-ID, MetFrag, MS-FINDER, SIRIUS) by matching features and counting consistent candidate metabolite/molecular formula assignments, reproducing the corroboration statistics of Fig. 4b–c.

## When to use

Use when a feature table carries network-propagated level 3 annotations (e.g. MetDNA3 output on Orbitrap data) and you need to quantify how many annotations independent in-silico tools support.

## When NOT to use

- Annotations are already level 1/2 confirmed with chemical standards — concordance testing adds no orthogonal information and cross-tool agreement would be circular
- Input features come from non-Orbitrap instruments not represented in the 20-dataset benchmark, so the 4,302-feature denominator and reported fractions do not apply
- You need de novo annotation rather than validation of existing network-based level 3 annotations

## Inputs

- Supplementary Data 3: MetDNA3 level 3 annotation table for 4,302 features from 20 Orbitrap datasets
- Source Data file for Fig. 4b–c: independent-tool annotation results (CFM-ID, MetFrag, MS-FINDER, SIRIUS) for the same feature set
- Feature identifier table mapping m/z and retention time across tools

## Outputs

- tool_corroboration_summary.csv with per-feature corroboration flags for each of the four tools
- Aggregate concordance counts/percentages: ≥1 tool consistent and all-four-tools consistent

## How to apply

Load Supplementary Data 3, containing MetDNA3 level 3 annotations for the 4,302 features from the 20 Orbitrap datasets, plus the independent-tool annotation results for the same feature set as provided in the Source Data file for Fig. 4b–c. Match features between MetDNA3 and each of CFM-ID, MetFrag, MS-FINDER, and SIRIUS by feature identifier (m/z / retention time), scoring a feature as corroborated when a tool annotates the same candidate metabolite or molecular formula. Then compute two concordance fractions: features with at least one consistent tool annotation (target: 3,894 of 4,302 = 90.1%) and features consistently supported by all four tools (target: 1,680 of 4,302 = 39.1%). Write per-feature corroboration flags plus aggregate counts/percentages to tool_corroboration_summary.csv. Rationale: the four tools operate without network propagation, so agreement provides an orthogonal check on the two-layer networking annotations, though the authors caution cross-tool concordance may overestimate performance because chemical standards are lacking.

## Related tools

- **MetDNA3** (Source of the network-based level 3 annotations being corroborated; implemented in MrnAnnoAlgo3 and the MetDNA3 webserver) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **CFM-ID** (Independent in-silico MS/MS tool used for cross evaluation of KGMN/level 3 annotations)
- **MetFrag** (Independent candidate-ranking annotation tool used for cross evaluation of KGMN/level 3 annotations)
- **MS-FINDER** (Independent formula/MS2 annotation tool used for cross evaluation of KGMN/level 3 annotations)
- **SIRIUS** (Independent isotope-pattern/MS2 annotation tool used for cross evaluation of KGMN/level 3 annotations)
- **MetDNA2** (Predecessor package whose README documents cross-evaluation tutorials for MetFrag, CFM-ID, and MS-FINDER used in this comparison workflow) — https://github.com/ZhuMetLab/MetDNA2

## Evaluation signals

- Features with ≥1 consistent tool annotation must equal 3,894 of 4,302 (90.1%)
- Features consistent across all four tools must equal 1,680 of 4,302 (39.1%)
- Denominator check: the level 3 feature count across the 20 Orbitrap datasets must be exactly 4,302
- Per-feature corroboration flags in tool_corroboration_summary.csv must be present for all four tools and logically consistent with the aggregate counts
- Matched features must align on m/z / retention time identifiers between MetDNA3 and each tool output

## Limitations

- Cross-tool concordance may overestimate performance due to the lack of chemical standards for the level 3 annotations
- Validation tools may have included part of the validation metabolites in their training, potentially contributing to the high accuracy
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so MS2-matching-based corroboration may be optimistic

## Evidence

- [readme] Cross evaluation of KGMN metabolites with MetFrag, CFM-ID, MS-FINDER: [evidence span withheld — anchor status: external_doc]
