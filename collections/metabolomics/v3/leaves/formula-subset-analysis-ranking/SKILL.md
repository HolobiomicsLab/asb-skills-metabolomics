---
name: formula-subset-analysis-ranking
description: Use when ranking candidate molecular formulas for MS/MS spectra of metabolites
  that may contain halogens (F, Cl, Br, I) beyond the validated CHONSP scope, and
  you need rank#1/top-5 accuracy and runtime benchmarks.
when_to_use_negative:
- The target compounds are lipids or contain elements outside CHONSP + F, Cl, Br,
  I — FSA has so far been validated only on C, H, O, N, S, P compounds, and halogen
  extension is a benchmarked extension, not the validated default scope
- You need definitive metabolite identification with structural information — FSA
  does not utilize structural information and its rank#1 accuracy is not yet comparable
  to state-of-the-art tools such as Buddy and SIRIUS; use it for fast prescreening
  instead
- You require an up-to-date chemical database — the FSA-packaged PubChem snapshot
  is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so counts and
  rankings may differ
edam_operation: http://edamontology.org/operation_3861
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FSA
  role: 'Formula ranking engine: generates and ranks candidate molecular formulas
    per MS/MS spectrum using the halogen-extended formula reference list'
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Source of molecular formulas and monoisotopic molecular weights used to build
    the expanded 2,613,203-entry reference formula list
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:e8b7adeb83c3629090441994ceded9fa73d4cb43aed9fc9a841c235188feab59
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
  edam_operation: http://edamontology.org/operation_3861
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# formula-subset-analysis-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply FSA (formula subset analysis) to rank candidate molecular formulas for LC-MS/MS spectra against an element-extended PubChem reference formula list (CHONSP plus F, Cl, Br, I = 2,613,203 entries), reporting rank#1 accuracy, top-5 accuracy, and per-spectrum runtime.

## When to use

Use when ranking candidate molecular formulas for MS/MS spectra of metabolites that may contain halogens (F, Cl, Br, I) beyond the validated CHONSP scope, and you need rank#1/top-5 accuracy and runtime benchmarks.

## When NOT to use

- The target compounds are lipids or contain elements outside CHONSP + F, Cl, Br, I — FSA has so far been validated only on C, H, O, N, S, P compounds, and halogen extension is a benchmarked extension, not the validated default scope
- You need definitive metabolite identification with structural information — FSA does not utilize structural information and its rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS; use it for fast prescreening instead
- You require an up-to-date chemical database — the FSA-packaged PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so counts and rankings may differ

## Inputs

- MS/MS spectra from non-lipid LC-MS/MS test datasets (peak lists per spectrum)
- PubChem compound database snapshot with molecular formulas and monoisotopic masses, filtered to formulas containing C, H, O, N, S, P and halogens F, Cl, Br, I (2,613,203 formulas)

## Outputs

- Ranked candidate molecular formula list per input spectrum
- Rank#1 accuracy and top-5 accuracy across the test datasets (expected ~77.84% and ~92.60% with halogens)
- Average runtime per spectrum (expected ~0.16 s with halogen-extended reference list)
- Structured comparison table of accuracy and runtime vs. reported benchmarks

## How to apply

Download a PubChem compound database snapshot and extract molecular formulas with their monoisotopic masses, retaining formulas that contain halogens (F, Cl, Br, I) in addition to C, H, O, N, S, P; this grows the reference list from 928,369 to 2,613,203 formulas. Configure FSA with this halogen-extended element/formula reference list and run it over the non-lipid MS/MS test datasets to produce ranked candidate molecular formulas per spectrum, recording wall-clock time per spectrum. Compute rank#1 accuracy, top-5 accuracy, and average runtime per spectrum from the FSA outputs, then compare against the reported halogen-inclusion benchmarks (77.84% rank#1, 92.60% top-5, 0.16 s/spectrum). The rationale is that FSA needs no training data and no MS/MS databases—only formulas and monoisotopic weights from public databases—so element scope can be expanded quickly, at the cost of runtime (~6.67x slower than the CHONSP-only 0.024 s/spectrum run).

## Related tools

- **FSA** (Formula ranking engine: generates and ranks candidate molecular formulas per MS/MS spectrum using the halogen-extended formula reference list) — https://github.com/kslynn128171/FSA
- **PubChem** (Source of molecular formulas and monoisotopic molecular weights used to build the expanded 2,613,203-entry reference formula list)

## Evaluation signals

- Reference formula count must equal 2,613,203 after including F, Cl, Br, I (vs 928,369 for CHONSP-only)
- Recomputed rank#1 accuracy across the six non-lipid test datasets falls within tolerance of 77.84%
- Recomputed top-5 accuracy falls within tolerance of 92.60%
- Average per-spectrum runtime falls within tolerance of 0.16 s (about 6.67x the CHONSP-only 0.024 s run)
- Comparison saved as a structured table matching the paper's reported halogen-inclusion metrics

## Limitations

- Runtime grows ~6.67x (0.024 s to 0.16 s per spectrum) when the reference list expands from CHONSP-only to halogen-inclusive
- Few chemical rules have been implemented to rank possible formulas and filter out unfeasible ones; such rules are crucial for certain compound species and adducts
- FSA has been validated only on datasets containing compounds composed of C, H, O, N, S, and P; halogen-containing and adduct-forming compounds are an extension, and rank#1 accuracy is not yet comparable to Buddy or SIRIUS
- The packaged PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), affecting candidate lists

## Evidence

- [results] halogen-extended reference list size and scope: [evidence span withheld — anchor status: not_found]
- [results] halogen-inclusion performance metrics: [evidence span withheld — anchor status: not_found]
