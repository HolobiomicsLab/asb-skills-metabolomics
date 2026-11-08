---
name: lc-ms-peak-annotation-prioritization
description: Use when untargeted LC-MS peak lists (mass-to-charge ratios with adducts/fragments)
  need to be annotated to KEGG compounds and a small prioritized proposal per peak
  is required, especially when standard annotation tools like xMSannotator underperform.
when_to_use_negative:
- When annotation against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB)
  is required — the article explicitly notes mWISE only annotates in the KEGG database,
  an important limitation relative to xMSannotator.
- When the input is already a curated annotated feature table with no need for candidate
  prioritization — the diffusion stage adds value only when multiple candidate compounds
  must be ranked per peak.
- When LC-MS peaks have not been pre-filtered (features with no signal or zero intensity
  not removed via the 80% rule) — the matching and clustering stages assume cleaned
  input peak lists.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0153
tools:
- name: mWISE
  role: 'R package implementing the full annotation pipeline: matching, cluster-based
    filtering and diffusion-based prioritization of LC-MS features to KEGG compounds'
  repo: https://dev.b2s.club/b2slab/mWISE
- name: Genomic Ranges
  role: Used by the matching stage to rapidly match all LC-MS peaks to the KEGG database
    considering a set of adducts and fragments
- name: Diffu Stats
  role: Called by set.diffusion to diffuse the label vector in the FELLA or RClass
    network
- name: FELLA
  role: Source of the biological network used in one of the mWISE diffusion configurations
- name: xMSannotator
  role: Baseline annotation algorithm that mWISE was benchmarked against (all four
    mWISE configurations outperform it in performance and computation time)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:a982e96787c9ee49f93668c32026dbd3e08c7817ae5505ca7252fdeb077a905e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8775510204081632
n_claims: 49
---

# lc-ms-peak-annotation-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A three-stage mWISE pipeline that matches LC-MS peak m/z values to the KEGG database, clusters and filters the candidate features to suppress false positives, and then diffuses score labels over a biochemical network (FELLA or RClass) to produce a per-peak prioritized annotation list.

## When to use

Use when untargeted LC-MS peak lists (mass-to-charge ratios with adducts/fragments) need to be annotated to KEGG compounds and a small prioritized proposal per peak is required, especially when standard annotation tools like xMSannotator underperform.

## When NOT to use

- When annotation against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) is required — the article explicitly notes mWISE only annotates in the KEGG database, an important limitation relative to xMSannotator.
- When the input is already a curated annotated feature table with no need for candidate prioritization — the diffusion stage adds value only when multiple candidate compounds must be ranked per peak.
- When LC-MS peaks have not been pre-filtered (features with no signal or zero intensity not removed via the 80% rule) — the matching and clustering stages assume cleaned input peak lists.

## Inputs

- Filtered LC-MS peak lists (m/z features, after removing features without signal and applying the 80% rule where intensity equals 0)
- KEGG compound database (via mWISE, considering a set of adducts and fragments)
- A biochemical or biological network: FELLA network (from the FELLA R package) or RClass network
- Reference annotation tables for benchmarking (TP/FP/FN/TN ground truth)

## Outputs

- Per-peak prioritized list of KEGG compounds (top three candidates per peak)
- Diffusion scores per candidate (z-normalized score with probability input, or raw score with binary input)
- Clustered/filtered feature sets reducing candidate false positives
- Benchmark metrics table: TP, FP, FN, TN, sensitivity, specificity, precision, accuracy, F1 per assay

## How to apply

Run the mWISE matching stage, which uses the GenomicRanges R package to match all LC-MS peaks to the KEGG database considering a set of adducts and fragments. Apply the cluster-based filter: features likely to originate from the same metabolite are grouped (spectral clustering, k-means, or DBSCAN) and the filter removes false-positive candidates before diffusion, which the article shows improves the final prioritization (e.g. Assay 2 FP dropped from 17598 to 1671). Compute the initial diffusion labels vector with diffusion.input and diffuse it with set.diffusion, which calls DiffuStats' diffuse() over either the FELLA biological network or the RClass biochemical network; scores are computed either with probability input type and the z-normalized score, or with binary input using the raw score and unique-annotation option. Select the top three candidates per peak, if available, as the final prioritized proposal, then compare those proposals against reference annotations to count TP, FP, FN and TN per assay and derive sensitivity, specificity, precision, accuracy and F1.

## Related tools

- **mWISE** (R package implementing the full annotation pipeline: matching, cluster-based filtering and diffusion-based prioritization of LC-MS features to KEGG compounds) — https://dev.b2s.club/b2slab/mWISE
- **Genomic Ranges** (Used by the matching stage to rapidly match all LC-MS peaks to the KEGG database considering a set of adducts and fragments)
- **Diffu Stats** (Called by set.diffusion to diffuse the label vector in the FELLA or RClass network)
- **FELLA** (Source of the biological network used in one of the mWISE diffusion configurations)
- **xMSannotator** (Baseline annotation algorithm that mWISE was benchmarked against (all four mWISE configurations outperform it in performance and computation time))

## Examples

```
Rscript mWISE_pipeline.R --assay assay6 --network RClass --score znorm --input peaks.csv --output rclass_znorm_metrics.csv
```

## Evaluation signals

- Matched-stage sensitivity should be near 0.92 for Assay 1 (144 TP, 13 FN, 157 references).
- After the cluster-based filter, false positives drop sharply (e.g. Assay 2: FP reduced from 17598 to 1671, specificity 0.91).
- Diffusion prioritization with the FELLA network and raw scores reaches F1 up to 0.49 (Assay 6, binary input).
- Using the RClass network with the z-normalized score and probability input yields sensitivities of 0.55-0.62 and F1 up to 0.39 (Assay 6: sensitivity 0.62, precision 0.29, F1 0.39).
- All four mWISE configurations must outperform xMSannotator in both performance and computation time (xMSannotator best F1 0.22 vs mWISE 0.49).

## Limitations

- mWISE only annotates against the KEGG database; support for HMDB, Lipid Maps, or T3DB (as in xMSannotator) is absent and should be addressed in future versions.
- Performance depends on prior input peak filtering (removing features without signal and applying the 80% rule where intensity equals 0).
- Only the top three candidates per peak are retained as the final prioritized proposal, which may discard correct lower-ranked annotations.
- The original GitHub repository (b2slab/mWISE) has been migrated; the active repository is now hosted on Forgejo at https://dev.b2s.club/b2slab/mWISE.

## Evidence

- [results] mWISE matching sensitivity for Assay 1 was 0.92 (144 TP, 13 FN, 157 references): [evidence span withheld — anchor status: not_found]
