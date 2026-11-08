---
name: halogen-element-formula-expansion
description: Use when MS/MS spectra may contain halogen-containing metabolites (F,
  Cl, Br, I) and the default FSA CHONSP reference list of 928,369 formulas is too
  narrow to rank the correct candidate formula.
when_to_use_negative:
- Samples are known to be limited to compounds composed of C, H, O, N, S, and P —
  the smaller CHONSP-only reference list is faster (0.024 s vs 0.16 s per spectrum)
  and more accurate (85.28% vs 77.84% rank#1).
- The goal is state-of-the-art rank#1 formula accuracy using structural information;
  the paper notes FSA's accuracy 'was not yet comparable to that of state-of-the-art
  tools such as Buddy and SIRIUS'.
- Working with compounds forming various adduct types — the paper states FSA has not
  been validated for such cases even with the expanded element set.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: FSA
  role: Formula-subset-analysis engine reconfigured with the halogen-extended reference
    formula list to rank candidate molecular formulas per MS/MS spectrum
  repo: kslynn128171/FSA
- name: PubChem
  role: Source of molecular formulas and monoisotopic masses used to build the expanded
    2,613,203-entry reference formula list
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:4409ba77e0a6b554086d1c2e4f150234ac2772abc86599b76ed2ff0bf06a73d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# halogen-element-formula-expansion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extends the FSA molecular-formula ranking engine beyond its validated CHONSP scope by rebuilding the PubChem-derived reference formula list with halogen elements (F, Cl, Br, I), trading rank#1 accuracy (85.28% → 77.84%) for broader metabolite coverage.

## When to use

Use when MS/MS spectra may contain halogen-containing metabolites (F, Cl, Br, I) and the default FSA CHONSP reference list of 928,369 formulas is too narrow to rank the correct candidate formula.

## When NOT to use

- Samples are known to be limited to compounds composed of C, H, O, N, S, and P — the smaller CHONSP-only reference list is faster (0.024 s vs 0.16 s per spectrum) and more accurate (85.28% vs 77.84% rank#1).
- The goal is state-of-the-art rank#1 formula accuracy using structural information; the paper notes FSA's accuracy 'was not yet comparable to that of state-of-the-art tools such as Buddy and SIRIUS'.
- Working with compounds forming various adduct types — the paper states FSA has not been validated for such cases even with the expanded element set.

## Inputs

- MS/MS spectra from LC-MS-based metabolomics runs (e.g., the six non-lipid test datasets used in the paper)
- PubChem compound database snapshot with molecular formulas and monoisotopic molecular weights
- FSA configuration with element/formula reference list enabled for F, Cl, Br, I in addition to C, H, O, N, S, P

## Outputs

- Ranked candidate molecular formula list per spectrum
- Per-spectrum wall-clock runtime log
- Structured table of rank#1 accuracy, top-5 accuracy, and average runtime versus the halogen-inclusion reference metrics

## How to apply

Download a PubChem compound database snapshot and extract molecular formulas with their monoisotopic masses, retaining formulas that contain halogens (F, Cl, Br, I) in addition to C, H, O, N, S, P; this grows the reference list from 928,369 to 2,613,203 entries. Configure FSA with the halogen-extended element/formula reference list and run it over the MS/MS test spectra to produce ranked candidate molecular formulas per spectrum, recording wall-clock time per spectrum. Compute rank#1 accuracy, top-5 accuracy, and average runtime per spectrum, then compare against the reported halogen-inclusion baselines of 77.84% rank#1, 92.60% top-5, and 0.16 s per spectrum. Expect roughly a 6.67x runtime slowdown relative to the CHONSP-only run (0.024 s/spectrum); save the accuracy/runtime comparison as a structured table.

## Related tools

- **FSA** (Formula-subset-analysis engine reconfigured with the halogen-extended reference formula list to rank candidate molecular formulas per MS/MS spectrum) — kslynn128171/FSA
- **PubChem** (Source of molecular formulas and monoisotopic masses used to build the expanded 2,613,203-entry reference formula list)

## Evaluation signals

- Reference formula list count must equal 2,613,203 entries after halogen inclusion (vs 928,369 CHONSP-only).
- Recomputed rank#1 accuracy across the six non-lipid test datasets should fall within tolerance of the reported 77.84%.
- Recomputed top-5 accuracy should fall within tolerance of the reported 92.60%.
- Average per-spectrum runtime should be near 0.16 s, i.e., about 6.67x the CHONSP-only 0.024 s/spectrum baseline.
- Per-spectrum wall-clock times must be recorded so the runtime comparison is reproducible.

## Limitations

- Expanding the element scope trades accuracy for coverage: rank#1 accuracy drops from 85.28% to 77.84% and runtime grows 6.67x (0.024 s → 0.16 s per spectrum).
- Few chemical rules are implemented to rank possible formulas and filter out unfeasible ones; such rules can be crucial for certain compound species and compounds with adducts.
- The embedded PubChem snapshot was outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so rebuilt reference lists will differ from the paper's counts with a newer snapshot.
- FSA does not utilize structural information, limiting its rank#1 accuracy relative to Buddy and SIRIUS.

## Evidence
