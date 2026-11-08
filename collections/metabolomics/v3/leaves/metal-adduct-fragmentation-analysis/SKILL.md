---
name: metal-adduct-fragmentation-analysis
description: Use when molecular networks contain ion variants annotated as [M+Na]+
  (or other metal adducts) that fail to link to their [M+H]+ counterparts under conventional
  similarity-threshold networks, and you need to verify whether the connectivity method
  recovers the true metabolite relationship.
when_to_use_negative:
- Input is already a feature table with adduct relationships resolved (e.g. via CAMERA-style
  annotation) — no network connectivity check is needed
- Spectra are compared within the same adduct class (e.g. [M+H]+ vs [M+H]+ only),
  where conventional cosine thresholds already suffice
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Python (pandas)
  role: Used to load the deposited spectral/network data, identify [M+Na]+/[M+H]+
    pairs, and assemble the verification table
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:47369c4907c02cc78b86577a24c04d1aec91e62ca6d4d6abf55c93e1af0e12ee
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Python
  - Python (pandas)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metal-adduct-fragmentation-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identifies and links metal-adducted ion variants (e.g. [M+Na]+) to their protonated ([M+H]+) precursors in molecular networks, where metal adduction shifts fragmentation patterns and breaks conventional cosine-similarity links. Grounded in the molecular community networking (MCN) workflow that recovers these otherwise-hidden metabolite connections.

## When to use

Use when molecular networks contain ion variants annotated as [M+Na]+ (or other metal adducts) that fail to link to their [M+H]+ counterparts under conventional similarity-threshold networks, and you need to verify whether the connectivity method recovers the true metabolite relationship.

## When NOT to use

- Input is already a feature table with adduct relationships resolved (e.g. via CAMERA-style annotation) — no network connectivity check is needed
- Spectra are compared within the same adduct class (e.g. [M+H]+ vs [M+H]+ only), where conventional cosine thresholds already suffice

## Inputs

- Spectral/network data files underlying Figures S12–S13 (deposited in the article repository)
- Ion-variant annotations identifying [M+Na]+ and [M+H]+ adduct forms

## Outputs

- verification_summary.csv listing pair ID, adduct forms, MCN link presence, and conventional-network link absence
- Flagged list of sodiated–protonated pairs where MCN fails to recover the expected link

## How to apply

Download the deposited spectral/network data from the article repository and load it with Python (pandas). Enumerate ion variants annotated as [M+Na]+ and match them to their putative [M+H]+ counterparts. For each pair, check whether MCN connects them while a conventional cosine-threshold molecular network does not, recording in-degree/connectivity evidence per pair. Assemble a verification table with pair ID, adduct forms, MCN link presence, and conventional-network link absence; save as verification_summary.csv and flag any pair where MCN fails to recover the link.

## Related tools

- **Python (pandas)** (Used to load the deposited spectral/network data, identify [M+Na]+/[M+H]+ pairs, and assemble the verification table) — https://github.com/Alexander0/molecular_communities

## Examples

```
import pandas as pd; df = pd.read_csv('fig_s12_network_links.csv'); pairs = df[(df.adduct_A=='[M+Na]+') & (df.adduct_B=='[M+H]+')]; pairs.to_csv('verification_summary.csv', index=False)
```

## Evaluation signals

- verification_summary.csv shows MCN link present for each sodiated–protonated pair while the conventional cosine-threshold network shows no link
- No flagged pair where MCN fails to recover the expected link (or flagged cases are explicitly explained by adduct-specific fragmentation differences)
- Each row in the summary has a non-null pair ID and adduct forms from the set {[M+Na]+, [M+H]+}
- Connectivity/in-degree values for MCN links are numerically consistent with the deposited network graph

## Limitations

- Spectra of sodiated metabolites remain more similar to their parent compounds than to any other molecules, so the method relies on MCN's connectivity optimization rather than raw spectral similarity alone
- Conventional molecular networking uses arbitrary similarity thresholds that can fracture natural metabolite families; this skill does not address threshold tuning itself, only adduct-link recovery

## Evidence

- [supplementary] Metal adduct alters fragmentation: [evidence span withheld — anchor status: not_found]
- [supplementary] MCN links sodiated variants to protonated precursors: [evidence span withheld — anchor status: not_found]
