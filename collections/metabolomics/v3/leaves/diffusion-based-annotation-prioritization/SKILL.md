---
name: diffusion-based-annotation-prioritization
description: Use when LC-MS peaks have already been matched to KEGG candidates and
  clustered, and a context-based prioritized list of compounds is needed to reduce
  false positives and rank annotations.
when_to_use_negative:
- Input is a raw peak list that has not yet been matched to KEGG candidates or clustered/filter
  via cluster Based.filter — run matching and clustering first.
- Annotation target is a database other than KEGG (e.g. HMDB, Lipid Maps, T3DB) —
  mWISE only offers annotation in the KEGG database, which the paper flags as an important
  limitation.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: mWISE
  role: R package whose set.diffusion function performs the diffusion-based prioritization
    over the KEGG candidate network
  repo: https://github.com/b2slab/mWISE
- name: Diffu Stats
  role: R package used by set.diffusion to diffuse the label vector in a given network
    and compute diffusion scores
- name: FELLA
  role: Supplies the biological network combined with raw scores in the article's
    best-performing diffusion configuration
- name: Genomic Ranges
  role: Used upstream by the matching Stage to rapidly match LC-MS peaks to KEGG candidates
    before diffusion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:bba4568567477fe4da37b45488f7f0cc51aac7078dd6bfcb6a839bbf323fefee
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 27
---

# diffusion-based-annotation-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prioritize KEGG compound candidates for LC-MS features by diffusing an initial label vector over a biochemical/biological graph (e.g. the FELLA network) with DiffuStats, producing diffusion scores that rank candidates per peak.

## When to use

Use when LC-MS peaks have already been matched to KEGG candidates and clustered, and a context-based prioritized list of compounds is needed to reduce false positives and rank annotations.

## When NOT to use

- Input is a raw peak list that has not yet been matched to KEGG candidates or clustered/filter via cluster Based.filter — run matching and clustering first.
- Annotation target is a database other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE only offers annotation in the KEGG database, which the paper flags as an important limitation.

## Inputs

- Peak list matched to KEGG candidates (set of LC-MS features with candidate compound assignments)
- Label vector from diffusion.input (binary or probability initial labels)
- Biological/biochemical network, e.g. from the FELLA R package or RClass

## Outputs

- Diffusion-based scores per KEGG candidate per LC-MS peak
- Prioritized ranked list of KEGG compounds (top three candidates per peak)

## How to apply

After the matching Stage has matched LC-MS peaks to KEGG candidates (GenomicRanges) and the cluster-based filter has reduced false positives, compute the initial diffusion labels vector with diffusion.input, then call set.diffusion to diffuse that vector via the DiffuStats R package over the chosen network — the paper's top configuration combines the FELLA biological network with raw scores. Score computation options are the probability input type with the z-normalized score, or binary input with raw score and the unique-annotation option; select the top three candidates per peak as the final prioritized proposal. This step improves specificity (e.g. Assay 2 false positives dropped from 17598 to 1671, specificity 0.91) and reaches F1 up to 0.49 with FELLA-raw diffusion (Assay 6), so its role is to convert a broad candidate list into a ranked, context-informed annotation.

## Related tools

- **mWISE** (R package whose set.diffusion function performs the diffusion-based prioritization over the KEGG candidate network) — https://github.com/b2slab/mWISE
- **Diffu Stats** (R package used by set.diffusion to diffuse the label vector in a given network and compute diffusion scores)
- **FELLA** (Supplies the biological network combined with raw scores in the article's best-performing diffusion configuration)
- **Genomic Ranges** (Used upstream by the matching Stage to rapidly match LC-MS peaks to KEGG candidates before diffusion)

## Examples

```
R: set.diffusion(diffusion.input(labels), network = FELLA::buildNetwork(), scores = "raw", input.type = "binary")
```

## Evaluation signals

- False positives decrease after applying the cluster-based filter before diffusion (Assay 2: FP falls from 17598 to 1671, specificity ≈ 0.91).
- Per-assay F1 from the diffusion prioritization reaches up to 0.49 with FELLA-raw (Assay 6) and 0.39 with RClass-raw; scores substantially below this suggest misconfiguration.
- Exactly the top three candidates per peak (when available) are returned as the final prioritized proposal.
- Diffusion runs use either probability input with z-normalized score or binary input with raw score and unique annotation; other option combinations are off-spec.
- Wall-clock runtime should be in the same order as the paper's mWISE FELLA-raw runs (≈15.7–53.4 minutes across the six assays), not the xMSannotator range (≈106–318 minutes).

## Limitations

- mWISE only annotates against the KEGG database; other resources (HMDB, Lipid Maps, T3DB) are not supported.
- Without the cluster-based filter upstream, diffusion introduces a large number of false positives (e.g. 17598 in Assay 2).
- Performance depends on the network and scoring configuration; alternative configurations (RClass vs FELLA, probability vs binary input) can shift sensitivity and specificity per assay.

## Evidence

- [methods] set.diffusion uses Diffu Stats: [evidence span withheld — anchor status: not_found]
