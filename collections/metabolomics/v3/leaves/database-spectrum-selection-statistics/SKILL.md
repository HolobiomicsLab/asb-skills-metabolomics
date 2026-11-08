---
name: database-spectrum-selection-statistics
description: Use when preparing HMDB, GNPS, or CASMI MS/MS spectral datasets for formula-ranking
  benchmarks and you must quantify how many spectra/metabolites survive the CHONSP,
  <1500 Da, precursor-type, and >=3-peak selection criteria.
when_to_use_negative:
- Compounds containing elements beyond C, H, O, N, S, and P, or forming adducts other
  than [M+H]+/[M-H]- — the criteria (and FSA itself) were validated only on CHONSP
  compounds with these precursors
- Spectra with fewer than three centroid peaks — they will be discarded by criterion
  4, so downstream ranking results would be meaningless
- When the goal is formula ranking of already-qualified spectra rather than dataset
  qualification statistics — use FSA ranking directly instead
edam_operation: http://edamontology.org/operation_3695
tools:
- name: ProteoWizard
  role: msconvert converts vendor-format HMDB/GNPS/CASMI raw files so each spectrum's
    formula, monoisotopic mass, precursor type, and centroid peak list can be parsed
    for the four-criteria filter
- name: FSA
  role: Downstream consumer of the qualified datasets; the filtered counts define
    the 30,690-spectrum evaluation set over which FSA reports top-1/top-5 formula
    accuracy
  repo: kslynn128171/FSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:88d8c484c8957d69664ffdccd11e6f8dcc540e12b05d8e3af35118b818623f52
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
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# database-spectrum-selection-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the paper's four dataset-qualification criteria (CHONSP-only formulas with mandatory C and H, mass < 1500 Da, [M+H]+/[M-H]- precursors, at least three peaks) to raw HMDB, GNPS, and CASMI MS/MS spectra, and reports per-dataset qualified spectra/compound counts to reproduce Table 1.

## When to use

Use when preparing HMDB, GNPS, or CASMI MS/MS spectral datasets for formula-ranking benchmarks and you must quantify how many spectra/metabolites survive the CHONSP, <1500 Da, precursor-type, and >=3-peak selection criteria.

## When NOT to use

- Compounds containing elements beyond C, H, O, N, S, and P, or forming adducts other than [M+H]+/[M-H]- — the criteria (and FSA itself) were validated only on CHONSP compounds with these precursors
- Spectra with fewer than three centroid peaks — they will be discarded by criterion 4, so downstream ranking results would be meaningless
- When the goal is formula ranking of already-qualified spectra rather than dataset qualification statistics — use FSA ranking directly instead

## Inputs

- Raw HMDB MS/MS spectra (HMDB_QTOF, HMDB_ITFT, HMDB_QFT) with chemical formula, monoisotopic mass, precursor type, and centroid peak list
- Raw GNPS MS/MS spectra (NPL_MAXIS, NPL_QTOF, NPL_OT, LIPID_OT) with formula, mass, precursor type, and peak list
- Raw CASMI spectra (CASMI_OT) with formula, mass, precursor type, and peak list

## Outputs

- Per-dataset table of qualified spectra and distinct compound counts before and after the four selection criteria (reproducing Table 1)
- Filtered spectrum collections passing all four criteria, ready for formula-ranking evaluation with FSA

## How to apply

Parse each raw spectrum (via msconvert in ProteoWizard for vendor-format conversion) and extract its chemical formula, monoisotopic mass, precursor type, and centroid peak list. Then sequentially apply the four criteria: (1) keep only formulas composed exclusively of C, H, O, N, S, and P with both C and H present; (2) keep only monoisotopic molecular weights < 1500 Da; (3) keep only [M+H]+ or [M-H]- precursor types; (4) keep only spectra with at least three centroid/deisotoped peaks. Count surviving spectra and distinct metabolites per dataset and tabulate them beside the raw totals. Validate against Table 1: HMDB 24,909 spectra/5,630 metabolites, GNPS 5,153/1,036, CASMI 628/362, with per-subset values such as HMDB_QTOF 3029 spectra (821 compounds) from 3743 (995) and LIPID_OT 18,084 (896).

## Related tools

- **ProteoWizard** (msconvert converts vendor-format HMDB/GNPS/CASMI raw files so each spectrum's formula, monoisotopic mass, precursor type, and centroid peak list can be parsed for the four-criteria filter)
- **FSA** (Downstream consumer of the qualified datasets; the filtered counts define the 30,690-spectrum evaluation set over which FSA reports top-1/top-5 formula accuracy) — kslynn128171/FSA

## Examples

```
msconvert sample.raw --mzML
```

## Evaluation signals

- Per-dataset qualified counts match Table 1 within the paper's stated numbers (e.g. HMDB_QTOF 3029 spectra / 821 compounds; HMDB_ITFT 2601/242; HMDB_QFT 1439/221; NPL_MAXIS 3561/2833; NPL_QTOF 1373/1360; NPL_OT 270/239; LIPID_OT 18,084/896; CASMI_OT 333/302)
- Aggregate totals equal HMDB 24,909 spectra / 5,630 metabolites, GNPS 5,153 / 1,036, CASMI 628 / 362
- Every retained formula contains only C, H, O, N, S, P atoms with both C and H present (zero violations after filtering)
- All retained spectra have monoisotopic mass < 1500 Da, precursor type in {[M+H]+, [M-H]-}, and >= 3 centroid peaks
- Before/after counts per dataset are tabulated so the reduction from raw to qualified spectra is auditable

## Limitations

- The qualification criteria restrict datasets to CHONSP compounds with protonated/deprotonated precursors, excluding halogen-, metal-containing compounds and other adduct species
- FSA itself has not yet incorporated chemical rules to filter unfeasible formulas or handle species/adduct-specific rules, so qualified datasets still contain ambiguous formula candidates
- The chemical-formula universe used for matching reflects a PubChem snapshot of 109,383,767 compounds, which is outdated relative to the 2022/11/27 version of 112,419,402 compounds

## Evidence
