---
name: molecular-networking-link-verification
description: Use when validating an MCN molecular network against a conventional similarity-threshold
  network by checking whether specific ion pairs (e.g. [M+Na]+/[M+H]+ adduct variants
  of the same metabolite) are connected only in the MCN graph.
when_to_use_negative:
- The network under test is already an MCN graph and you only need descriptive statistics
  rather than a link-verification comparison against a conventional network.
- The ion pairs of interest are not adduct variants of the same metabolite (e.g. unrelated
  metabolites); the skill is specifically scoped to sodiated–protonated adduct pairs.
- No conventional similarity-threshold network is available for comparison — the verification
  depends on contrasting MCN links with conventional-network absence.
edam_operation: http://edamontology.org/operation_3503
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Python
  role: Used to load the spectral/network data, identify ion variants annotated as
    [M+Na]+ and their [M+H]+ counterparts, and assemble the verification table.
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:27a770e0720b39c7856ad08d9dc00e139a0b3c5686b9af51103c9c3addff3e0c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_3503
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecular-networking-link-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verification workflow for molecular community networking (MCN) that tests whether adduct variants (e.g. sodiated [M+Na]+ ions) are linked to their protonated [M+H]+ precursors in MCN output but remain disconnected in conventional threshold-based molecular networks.

## When to use

Use when validating an MCN molecular network against a conventional similarity-threshold network by checking whether specific ion pairs (e.g. [M+Na]+/[M+H]+ adduct variants of the same metabolite) are connected only in the MCN graph.

## When NOT to use

- The network under test is already an MCN graph and you only need descriptive statistics rather than a link-verification comparison against a conventional network.
- The ion pairs of interest are not adduct variants of the same metabolite (e.g. unrelated metabolites); the skill is specifically scoped to sodiated–protonated adduct pairs.
- No conventional similarity-threshold network is available for comparison — the verification depends on contrasting MCN links with conventional-network absence.

## Inputs

- MCN molecular network edge list (deposited data underlying Figures S12–S13)
- Conventional molecular network edge list with similarity-threshold cutoffs
- Node annotation table mapping metabolite IDs to adduct labels ([M+Na]+, [M+H]+)
- Python 3 environment with pandas

## Outputs

- verification_summary.csv — per-pair table with columns: pair ID, adduct forms, MCN link presence, conventional-network link absence
- Flagged pair list of sodiated–protonated pairs where MCN fails to recover the link

## How to apply

Load the MCN network output and the conventional molecular network (both derived from the article's deposited data, e.g. Figures S12–S13) into pandas DataFrames. Annotate ion variants by their adduct label — identify nodes flagged as [M+Na]+ and their putative protonated [M+H]+ counterparts of the same metabolite. For each sodiated–protonated pair, query the MCN edge list for a direct or indirect connection while querying the conventional similarity-threshold network for the absence of such a link; this exploits the paper's claim that metal adduction alters fragmentation so spectra of sodiated metabolites are more similar to the parent compounds than to other molecules. Assemble the evidence into a per-pair verification table recording pair ID, adduct forms, MCN link presence, and conventional-network link absence. Save the table as verification_summary.csv and flag any pair where MCN fails to recover the expected link so it can be reviewed manually.

## Related tools

- **Python** (Used to load the spectral/network data, identify ion variants annotated as [M+Na]+ and their [M+H]+ counterparts, and assemble the verification table.) — https://github.com/Alexander0/molecular_communities

## Examples

```
import pandas as pd; edges_mcn = pd.read_csv('mcn_edges.csv'); edges_conv = pd.read_csv('conventional_edges.csv'); pairs = pd.read_csv('sodiated_protonated_pairs.csv'); pairs['mcn_link'] = pairs.apply(lambda r: ((edges_mcn.source==r.na_node)&(edges_mcn.target==r.h_node)).any(), axis=1); pairs['conv_link'] = pairs.apply(lambda r: ((edges_conv.source==r.na_node)&(edges_conv.target==r.h_node)).any(), axis=1); pairs.to_csv('verification_summary.csv', index=False)
```

## Evaluation signals

- verification_summary.csv exists and contains a row for each sodiated–protonated pair identified from the network annotations.
- For every pair in the summary, the MCN link column is TRUE while the conventional-network link column is FALSE, matching the paper's claim that conventional networks fracture these relationships via arbitrary similarity thresholds.
- Any pair where MCN fails to recover the link is explicitly flagged rather than silently omitted.
- Adduct labels in the summary match the source node annotations exactly (e.g. [M+Na]+ and [M+H]+ strings).

## Limitations

- Spectra of sodiated metabolites are still more similar to the parent compounds than to any other molecules, so the verification relies on the MCN optimisation rescuing these connections rather than a guaranteed structural rule.
- Conventional molecular networking uses arbitrary similarity thresholds, so the baseline network's absence of links is cutoff-dependent and must be reported alongside the thresholds used.

## Evidence

- [other] Are sodiated ion variants of metabolites, which are disconnected from their protonated precursors in conventional molecular networks because metal adduction alters fragmentation, linked by MCN?: [evidence span withheld — anchor status: not_found]
- [other] MCN links sodiated ion variants to their corresponding protonated precursors, even though spectra of sodiated metabolites are still more similar to the parent compounds than to any other molecules.: [evidence span withheld — anchor status: not_found]
