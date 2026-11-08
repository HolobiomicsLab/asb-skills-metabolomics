---
name: adduct-ion-variant-matching
description: Use when validating a molecular network's ability to recover adduct-related
  spectral pairs — specifically sodiated variants disconnected from protonated precursors
  under similarity-threshold networking.
when_to_use_negative:
- When the goal is generic spectral similarity clustering rather than explicitly testing
  adduct-variant linkage recovery
- When input is raw mass-spectrometry data rather than already-annotated network/spectral
  files — this skill operates on deposited network outputs, not on mzML acquisition
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the deposited spectral/network files and identifies [M+Na]+ / [M+H]+
    ion-variant pairs and their connectivity
- name: molecular_communities
  role: Source repository for the deposited data underlying the sodiated-variant linking
    analysis (Figures S12-13)
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:3cdefc59809e51eb125f14ccc34d73f9631984d2292b843df1700af4215d17fc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Python
  - molecular_communities
  - pandas
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# adduct-ion-variant-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verifies that molecular community networking (MCN) links metal-adduct ion variants (e.g., [M+Na]+) to their protonated [M+H]+ counterparts, relationships that conventional cosine-threshold molecular networks miss because adduction alters fragmentation patterns.

## When to use

Use when validating a molecular network's ability to recover adduct-related spectral pairs — specifically sodiated variants disconnected from protonated precursors under similarity-threshold networking.

## When NOT to use

- When the goal is generic spectral similarity clustering rather than explicitly testing adduct-variant linkage recovery
- When input is raw mass-spectrometry data rather than already-annotated network/spectral files — this skill operates on deposited network outputs, not on mzML acquisition

## Inputs

- Deposited spectral/network data files underlying Figures S12-S13 from the article's repository
- Ion annotations labeling adduct forms ([M+Na]+, [M+H]+)

## Outputs

- verification_summary.csv with columns: pair ID, adduct forms, MCN link presence, conventional-network link absence
- Flags for sodiated-protonated pairs where MCN fails to recover the link

## How to apply

Download the deposited spectral/network files underlying the supplementary figures from the article's repository (Alexander0/molecular_communities). Load the data with Python/pandas and identify ion variants annotated as [M+Na]+ together with their putative protonated [M+H]+ counterparts. For each pair, check whether MCN connects them while conventional similarity-threshold networks do not — the rationale is that metal adduction alters fragmentation, so spectra of sodiated metabolites are more similar to their parent compounds than to other molecules, yet threshold networks fracture these relationships. Record in-degree/connectivity evidence per pair into a verification table with pair ID, adduct forms, MCN link presence, and conventional-network link absence. Flag any pairs where MCN fails to recover the link before saving the summary.

## Related tools

- **pandas** (Loads the deposited spectral/network files and identifies [M+Na]+ / [M+H]+ ion-variant pairs and their connectivity)
- **molecular_communities** (Source repository for the deposited data underlying the sodiated-variant linking analysis (Figures S12-13)) — https://github.com/Alexander0/molecular_communities

## Examples

```
import pandas as pd; df = pd.read_csv('network_edges.csv'); pairs = df[(df.adduct_A=='[M+Na]+') & (df.adduct_B=='[M+H]+')]; pairs.to_csv('verification_summary.csv')
```

## Evaluation signals

- verification_summary.csv shows MCN links present for every sodiated-protonated pair tested
- Conventional similarity-threshold networks show no link for the same pairs
- No pairs are flagged where MCN fails to recover the link (or flags are justified)
- Each table row contains both adduct annotations ([M+Na]+ and [M+H]+) and a pair ID

## Limitations

- Conventional molecular networking uses arbitrary similarity thresholds that can fracture natural metabolite families — absence of a link there is expected, not a data error
- Metal adduction alters fragmentation, so sodiated spectra remain more similar to parent compounds than to other molecules; matching depends on correct adduct annotation of the deposited data
- Adduction effects may be compound- or adduct-specific; results verified for [M+Na]+/[M+H]+ pairs may not generalize to all adduct types

## Evidence

- [supplementary] MCN links sodiated ion variants to their corresponding protonated precursors: [evidence span withheld — anchor status: not_found]
- [abstract] Conventional molecular networking uses arbitrary similarity thresholds that can fracture natural metabolite families: [evidence span withheld — anchor status: not_found]
- [abstract] MCN captures hidden metabolite connections: "We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections."
