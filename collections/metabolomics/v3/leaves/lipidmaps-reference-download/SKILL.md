---
name: lipidmaps-reference-download
description: Use when the goal is to quantify overlap between CMM 3.0's in-house oxidized
  lipid library and public lipid databases, or to establish whether CMM-exclusive
  oxidized lipid annotations are genuinely novel.
when_to_use_negative:
- The analysis targets general (non-oxidized) metabolite annotation scoring, which
  CMM 2.0/3.0 handles with its expert-system filter rather than a library-overlap
  comparison
- No public reference database subset (e.g. LIPID MAPS oxidized phospholipids) is
  available, making the presence/absence comparison undefined
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Source of the in-house oxidized lipid library entries sampled for the overlap
    analysis
  repo: albertogilf/ceuMassMediator
- name: CMM RESTful API
  role: Programmatic retrieval of the sampled oxidized lipid entries from CMM 3.0
- name: LIPID MAPS
  role: Public reference database supplying the oxidized phospholipid subset to compare
    against
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:2fa95fd162d317188aed18db1a22ddeb6f28f45dc59b007585f9001d5cf877cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  repo_url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  - CMM RESTful API
  - LIPID MAPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lipidmaps-reference-download

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a public reference set of oxidized phospholipids from LIPID MAPS and cross-checks the CMM 3.0 in-house oxidized lipid library against it, quantifying how many CMM oxidized entries (e.g. oxidized glycerophosphocholines) are absent from public sources.

## When to use

Use when the goal is to quantify overlap between CMM 3.0's in-house oxidized lipid library and public lipid databases, or to establish whether CMM-exclusive oxidized lipid annotations are genuinely novel.

## When NOT to use

- The analysis targets general (non-oxidized) metabolite annotation scoring, which CMM 2.0/3.0 handles with its expert-system filter rather than a library-overlap comparison
- No public reference database subset (e.g. LIPID MAPS oxidized phospholipids) is available, making the presence/absence comparison undefined

## Inputs

- Oxidized glycerophosphocholine (PC) species identifiers/abbreviated names retrieved from the CMM 3.0 RESTful API
- Oxidized phospholipid subset downloaded from LIPID MAPS

## Outputs

- Per-entry comparison table oxidized_lipid_overlap.csv (each CMM entry marked present/absent in LIPID MAPS)
- Tallies of overlapping vs non-overlapping oxidized lipid entries

## How to apply

Retrieve a fixed sample of oxidized lipid entries served by CEU Mass Mediator (CMM) 3.0 through its RESTful API, focusing on oxidized glycerophosphocholine (PC) species. Download the oxidized phospholipid subset from LIPID MAPS as the public reference. Normalize lipid identifiers and abbreviated names on both sides (PC class and oxidation-level nomenclature) so that entries match across sources. Classify each sampled CMM entry as present or absent in the LIPID MAPS reference and tally overlapping vs non-overlapping counts. Write the per-entry comparison table to oxidized_lipid_overlap.csv; a high non-overlap tally supports the paper's claim that the in-house oxidized lipid library is not present in other sources.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Source of the in-house oxidized lipid library entries sampled for the overlap analysis) — albertogilf/ceuMassMediator
- **CMM RESTful API** (Programmatic retrieval of the sampled oxidized lipid entries from CMM 3.0)
- **LIPID MAPS** (Public reference database supplying the oxidized phospholipid subset to compare against)

## Evaluation signals

- Every sampled CMM oxidized lipid entry appears in oxidized_lipid_overlap.csv with a present/absent verdict against the LIPID MAPS reference
- Counts of overlapping and non-overlapping entries sum to the number of sampled CMM entries (no entries dropped)
- Identifier/abbreviated-name normalization applied consistently: PC class and oxidation-level nomenclature on both CMM and LIPID MAPS sides
- Overlapping entries can be traced to a concrete LIPID MAPS record; non-overlapping claims spot-checked against the downloaded reference subset

## Limitations

- Novelty conclusions depend on identifier/name normalization: unmatched entries may reflect nomenclature differences rather than true absence from public sources
- Sampling is limited to a fixed sample of oxidized glycerophosphocholine (PC) species, so non-overlap counts do not cover all oxidized lipid classes
- Deployment of CMM locally requires access to an internal CEU database requested by email and a configured app-server data-source, which can block direct reuse of the library

## Evidence
