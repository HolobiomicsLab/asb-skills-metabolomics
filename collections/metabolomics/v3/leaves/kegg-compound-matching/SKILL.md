---
name: kegg-compound-matching
description: Use when you have an untargeted LC-MS peak list (m/z, retention time,
  intensity) and need to map each peak to candidate KEGG compounds via adduct/fragment-aware
  m/z matching before annotation prioritization.
when_to_use_negative:
- You need to annotate against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB)
  — mWISE only offers the data to annotate in the KEGG database
- Input is already a set of candidate compounds with features (i.e. downstream of
  matchingStage) — use featuresClustering/clusterBased.filter instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: mWISE
  role: R package providing the matchingStage command used to match LC-MS peaks to
    the KEGG database
  repo: https://github.com/b2slab/mWISE
- name: Genomic Ranges
  role: R package underpinning the rapid peak-to-KEGG interval matching in matchingStage
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:f784ad2244f46351b7f7cb7034e399d25cbe15673e454c437c7acf0235c812be
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 30
---

# kegg-compound-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mWISE's matchingStage rapidly matches LC-MS peak m/z values to the KEGG compound database, considering a configurable set of adducts and fragments, producing per-peak candidate KEGG compounds that feed the subsequent clustering/filter and diffusion prioritization steps.

## When to use

Use when you have an untargeted LC-MS peak list (m/z, retention time, intensity) and need to map each peak to candidate KEGG compounds via adduct/fragment-aware m/z matching before annotation prioritization.

## When NOT to use

- You need to annotate against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE only offers the data to annotate in the KEGG database
- Input is already a set of candidate compounds with features (i.e. downstream of matchingStage) — use featuresClustering/clusterBased.filter instead

## Inputs

- Untargeted LC-MS peak list (m/z, retention time, intensity), optionally organized as a list with positive acquisition mode objects and another list with negative acquisition mode objects
- Set of allowed adducts and fragments for the ionization mode
- KEGG compound database entries (m/z-relevant mass and formula metadata)

## Outputs

- Per-peak candidate KEGG compound matches (matched stage output ready for featuresClustering)
- Per-assay confusion counts (TP/FP/FN) against reference compounds if benchmarked with performanceEvaluation

## How to apply

Run the mWISE matchingStage command, which is built on the GenomicRanges R package to rapidly match all LC-MS peaks to the KEGG database while considering a set of adducts and fragments. mWISE only offers the data to annotate in the KEGG database, so inputs should be m/z peak lists from untargeted LC-MS experiments (e.g. the Trypanosoma positive/negative acquisition mode lists). For each peak, matchingStage emits the candidate KEGG compounds whose adduct/fragment-derived masses fall within the matching tolerance. Matched candidates from multiple peaks are then clustered (featuresClustering) and filtered (clusterBased.filter) to reduce false positives before the diffusion step, but matchingStage itself is the first, standalone stage. Verify the stage by checking per-assay counts of true positives, false positives and false negatives against reference annotations — e.g. mWISE reported matching sensitivity of 0.92 for Assay 1 (144 TP, 13 FN, 157 references).

## Related tools

- **mWISE** (R package providing the matchingStage command used to match LC-MS peaks to the KEGG database) — https://github.com/b2slab/mWISE
- **Genomic Ranges** (R package underpinning the rapid peak-to-KEGG interval matching in matchingStage)

## Examples

```
matchingStage(peakList, adducts = setOfAdducts, fragments = setOfFragments)
```

## Evaluation signals

- Benchmark with mWISE's performanceEvaluation against a reference data frame: reported sensitivity for the matching stage was 0.92 for Assay 1 (144 TP, 13 FN, 157 references)
- Per-peak candidate set should be non-empty for peaks whose m/z falls within tolerance of at least one KEGG adduct/fragment mass
- Matching counts should be traceable in a confusion-matrix-style table (TP, FP, FN) per assay

## Limitations

- mWISE only offers the data to annotate in the KEGG database — a limitation relative to multi-database annotators like xMSannotator
- Matching quality depends on the chosen adduct/fragment set and on correct handling of both positive and negative acquisition mode objects

## Evidence

- [methods] matchingStage uses GenomicRanges for rapid KEGG matching: [evidence span withheld — anchor status: not_found]
