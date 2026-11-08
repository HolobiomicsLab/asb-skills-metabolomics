---
name: metabolite-annotation-confidence-level-matching
description: Use when validating network-derived (e.g., MetDNA3 level 3) metabolite
  annotations by testing concordance with independent tools such as CFM-ID, MetFrag,
  MS-FINDER, and SIRIUS on feature-matched annotations.
when_to_use_negative:
- Annotations already confirmed by chemical standards (level 1/2 confidence) — corroboration
  statistics target network-propagated level 3 annotations that lack standard confirmation
- The goal is de novo metabolite annotation rather than validating existing network-based
  annotation results with independent tools
- Tool outputs cannot be feature-matched by m/z / retention time because they were
  generated on a different feature set or raw data preprocessing
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3
  role: Source of network-based level 3 annotations being corroborated; implemented
    in the MrnAnnoAlgo3 R package and MetDNA3 webserver
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: CFM-ID
  role: Independent non-network-based in-silico MS/MS annotation tool used for cross
    evaluation of network-annotated metabolites
- name: MetFrag
  role: Independent candidate-ranking annotation tool used for cross evaluation of
    network-annotated metabolites
- name: MS-FINDER
  role: Independent formula/structure annotation tool used for cross evaluation of
    network-annotated metabolites
- name: SIRIUS
  role: Independent computational annotation tool used for cross evaluation of network-annotated
    metabolites
- name: GNPS
  role: Molecular networking ecosystem whose MS2 spectral similarity networks provide
    comparison results for the corroboration statistics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:8a82a18476e9226373908a5757462a241960be78bce4c28599fdad567032bf03
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolite-annotation-confidence-level-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corroborates network-based metabolite annotations (e.g., MetDNA3 level 3 annotations) by matching features against independent, non-network-based annotation tools and computing multi-tool concordance statistics, as done for Fig. 4b–c of the MetDNA3 paper.

## When to use

Use when validating network-derived (e.g., MetDNA3 level 3) metabolite annotations by testing concordance with independent tools such as CFM-ID, MetFrag, MS-FINDER, and SIRIUS on feature-matched annotations.

## When NOT to use

- Annotations already confirmed by chemical standards (level 1/2 confidence) — corroboration statistics target network-propagated level 3 annotations that lack standard confirmation
- The goal is de novo metabolite annotation rather than validating existing network-based annotation results with independent tools
- Tool outputs cannot be feature-matched by m/z / retention time because they were generated on a different feature set or raw data preprocessing

## Inputs

- Supplementary Data 3: MetDNA3 level 3 annotation table for 4,302 features across 20 Orbitrap datasets
- Independent-tool annotation results for the same feature set (CFM-ID, MetFrag, MS-FINDER, SIRIUS; GNPS/FBMN/IIMN/NetID results from the Source Data file for Fig. 4b–c)
- Feature identifiers (m/z and retention time) shared between MetDNA3 and tool outputs

## Outputs

- Per-feature corroboration flags (consistent/inconsistent per tool)
- Aggregate concordance counts and percentages (e.g., 3,894/4,302 = 90.1% with ≥1 tool; 1,680/4,302 = 39.1% with all four tools)
- tool_corroboration_summary.csv reproducing the statistics underlying Fig. 4b–c

## How to apply

Load the Supplementary Data 3 table of MetDNA3 level 3 annotations for the 4,302 features from the 20 Orbitrap datasets, then load the independent-tool annotation results for the same feature set (from the Source Data file underlying Fig. 4b–c, covering CFM-ID, MetFrag, MS-FINDER, SIRIUS, and the GNPS/FBMN/IIMN/NetID ecosystem results). Match features between MetDNA3 and each tool by feature identifier (m/z / retention time), and flag a feature as corroborated by a tool when that tool annotates the same candidate metabolite or molecular formula. Compute two aggregate statistics: the fraction of features with at least one consistent tool annotation (expected 3,894 of 4,302 = 90.1%) and the fraction consistently supported by all four tools (expected 1,680 of 4,302 = 39.1%). Write the per-feature corroboration flags and aggregate counts/percentages to tool_corroboration_summary.csv. Rationale: cross-tool concordance provides an independent check on network-propagated (level 3) annotations that lack chemical-standard confirmation, since MetDNA3's propagation relies on MS2 spectral similarity that cannot resolve isomers.

## Related tools

- **MetDNA3** (Source of network-based level 3 annotations being corroborated; implemented in the MrnAnnoAlgo3 R package and MetDNA3 webserver) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **CFM-ID** (Independent non-network-based in-silico MS/MS annotation tool used for cross evaluation of network-annotated metabolites)
- **MetFrag** (Independent candidate-ranking annotation tool used for cross evaluation of network-annotated metabolites)
- **MS-FINDER** (Independent formula/structure annotation tool used for cross evaluation of network-annotated metabolites)
- **SIRIUS** (Independent computational annotation tool used for cross evaluation of network-annotated metabolites)
- **GNPS** (Molecular networking ecosystem whose MS2 spectral similarity networks provide comparison results for the corroboration statistics)

## Examples

```
Rscript -e 'library(readr); d <- read_csv("Supplementary_Data_3.csv"); n <- nrow(d); write_csv(dplyr::tibble(n=n, any_tool=sum(d$any_consistent)/n, all4=sum(d$all_consistent)/n), "tool_corroboration_summary.csv")'
```

## Evaluation signals

- The count of features with at least one consistent tool annotation must equal 3,894 of 4,302 (90.1%)
- The count of features consistently supported by all four tools must equal 1,680 of 4,302 (39.1%)
- The feature denominator must remain 4,302 level 3 annotations across the 20 Orbitrap datasets (no features silently dropped during matching)
- Each per-feature corroboration flag must be a well-defined binary/boolean value per tool, with all four tool columns populated for every matched feature
- Recomputed concordance fractions must match the reported 90.1% (≥1 tool) and 39.1% (all four tools) exactly to reproduce Fig. 4b–c

## Limitations

- Cross-tool concordance may overestimate performance because chemical standards are lacking for the level 3 annotations
- Validation tools may have included part of the validation metabolites in their training data, potentially contributing to artificially high accuracy
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so concordance checks relying on MS2 spectral matching may be optimistic

## Evidence

- [readme] It also support the connection with other metabolomics workflow, like MetFrag, MS-FINDER, MASST etc.: [evidence span withheld — anchor status: external_doc]
