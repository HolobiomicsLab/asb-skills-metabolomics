---
name: cross-tool-annotation-concordance-analysis
description: Use when you have a table of network-based (MetDNA3 level 3) feature
  annotations plus independent tool annotations for the same features and need to
  compute per-tool or all-tools concordance percentages.
when_to_use_negative:
- Your annotations are level 1/2 with chemical standards rather than network-based
  level 3 annotations — concordance is not the intended validation mode there.
- You lack independent non-network tool annotation outputs for the same features (no
  CFM-ID/MetFrag/MS-FINDER/SIRIUS results), so corroboration cannot be computed.
- Your goal is to run or reproduce MetDNA3's annotation pipeline itself (use the MrnAnnoAlgo3/MetDNA2
  packages) rather than to evaluate agreement of existing annotation tables.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetDNA3
  role: Source of the network-based level 3 annotations being cross-evaluated (4,302
    features, 20 Orbitrap datasets)
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: CFM-ID
  role: Independent non-network in-silico MS/MS annotation tool used for corroboration
    of MetDNA3 level 3 annotations
  repo: https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- name: MetFrag
  role: Independent non-network in-silico fragmentation annotation tool used for corroboration
    of MetDNA3 level 3 annotations
  repo: https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- name: MS-FINDER
  role: Independent non-network formula/MS2 annotation tool used for corroboration
    of MetDNA3 level 3 annotations
  repo: https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- name: SIRIUS
  role: Independent non-network annotation tool used for corroboration of MetDNA3
    level 3 annotations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:2915ea3917088a47222eadf1b56d216d97f6bd50b909982bef40f411347311e8
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
  - MetDNA3
  - MetFrag
  - NetID
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-tool-annotation-concordance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many network-based (MetDNA3) level 3 metabolite annotations can be independently corroborated by non-network annotation tools (CFM-ID, MetFrag, MS-FINDER, SIRIUS) by matching candidates per feature and computing concordance fractions, as done for Fig. 4b–c of the MetDNA3 paper.

## When to use

Use when you have a table of network-based (MetDNA3 level 3) feature annotations plus independent tool annotations for the same features and need to compute per-tool or all-tools concordance percentages.

## When NOT to use

- Your annotations are level 1/2 with chemical standards rather than network-based level 3 annotations — concordance is not the intended validation mode there.
- You lack independent non-network tool annotation outputs for the same features (no CFM-ID/MetFrag/MS-FINDER/SIRIUS results), so corroboration cannot be computed.
- Your goal is to run or reproduce MetDNA3's annotation pipeline itself (use the MrnAnnoAlgo3/MetDNA2 packages) rather than to evaluate agreement of existing annotation tables.

## Inputs

- MetDNA3 level 3 annotation table for 4,302 features from the 20 Orbitrap datasets (Supplementary Data 3)
- Independent-tool annotation results (CFM-ID, MetFrag, MS-FINDER, SIRIUS) for the same feature set, from the Source Data file for Fig. 4b–c
- Feature identifiers (m/z / retention time) common to both annotation tables

## Outputs

- Per-feature corroboration flags (one boolean per tool per feature)
- tool_corroboration_summary.csv with aggregate counts/percentages (at-least-one-tool and all-four-tools concordance)

## How to apply

Load the MetDNA3 level 3 annotation table (Supplementary Data 3, 4,302 features from the 20 Orbitrap datasets) into pandas or readr. Load the independent-tool annotation results for the same feature set from the Source Data file for Fig. 4b–c. Match features between MetDNA3 and each tool by feature identifier (m/z / retention time) and mark a feature as corroborated by a tool when that tool annotates the same candidate metabolite or molecular formula. Build per-feature boolean corroboration flags for CFM-ID, MetFrag, MS-FINDER, and SIRIUS, then compute (a) the fraction with at least one consistent tool annotation and (b) the fraction consistently supported by all four tools. The rationale is that these four tools are non-network-based, so concordance provides an independent (though imperfect, see limitations) check on network-based annotation propagation.

## Related tools

- **MetDNA3** (Source of the network-based level 3 annotations being cross-evaluated (4,302 features, 20 Orbitrap datasets)) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **CFM-ID** (Independent non-network in-silico MS/MS annotation tool used for corroboration of MetDNA3 level 3 annotations) — https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- **MetFrag** (Independent non-network in-silico fragmentation annotation tool used for corroboration of MetDNA3 level 3 annotations) — https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- **MS-FINDER** (Independent non-network formula/MS2 annotation tool used for corroboration of MetDNA3 level 3 annotations) — https://github.com/ZhuMetLab/MetDNA2_Web/blob/main/Tutorials/Tutorial_KGMN_and_insilico_ms2.pdf
- **SIRIUS** (Independent non-network annotation tool used for corroboration of MetDNA3 level 3 annotations)

## Examples

```
import pandas as pd; df = pd.read_csv('Supplementary_Data_3.csv'); flags = df[['cfmid','metfrag','msfinder','sirius']]; print(flags.sum(axis=1).ge(1).mean(), flags.all(axis=1).mean())
```

## Evaluation signals

- The ≥1-tool concordance fraction must equal 3,894 / 4,302 = 90.1%.
- The all-four-tools concordance fraction must equal 1,680 / 4,302 = 39.1%.
- The denominator must be exactly 4,302 level 3 features across the 20 Orbitrap datasets — any other denominator indicates a feature-matching error.
- The per-feature corroboration flags must be boolean and complete (no missing values) for all four tools before aggregating.
- tool_corroboration_summary.csv must reproduce the per-feature flags and the aggregate counts underlying Fig. 4b–c.

## Limitations

- Cross-tool concordance may overestimate performance due to the lack of chemical standards for the level 3 annotations.
- The validation tools may have included part of the validation metabolites in their training, potentially contributing to inflated accuracy.
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so agreement based on MS2 spectral matching may be optimistic.

## Evidence

- [results] Of 4,302 features with level 3 annotations across the 20 Orbitrap datasets, 3,894 (90.1%) had at least one consistent annotation between MetDNA3 and one of the four tools: [evidence span withheld — anchor status: not_found]
- [discussion] cross-tool concordance may overestimate performance due to the lack of chemical standards: "cross-tool concordance may overestimate performance due to the lack of chemical standards"
- [readme] Cross evaluation of KGMN metabolites: [evidence span withheld — anchor status: external_doc]
