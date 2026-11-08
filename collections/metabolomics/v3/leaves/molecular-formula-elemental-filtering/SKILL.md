---
name: molecular-formula-elemental-filtering
description: Use when preparing curated benchmark MS/MS datasets from HMDB, GNPS,
  or CASMI spectra and you must qualify entries by elemental composition, mass, precursor
  type, and peak count.
when_to_use_negative:
- Spectra whose precursor type is anything other than [M+H]+ or [M-H]-, since these
  are excluded by criterion 3
- Compound libraries containing elements beyond C, H, O, N, S, and P — the paper explicitly
  has not validated FSA on molecules outside this element set
- Compounds with monoisotopic mass >= 1500 Da, which criterion 2 removes
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ProteoWizard
  role: msconvert is used as the free software to convert/prepare the raw MS/MS spectral
    source files before parsing formulas, masses, precursors, and peak lists for filtering
- name: FSA
  role: Downstream consumer of the eight qualified datasets; the filter builds the
    benchmark on which FSA generates and ranks candidate molecular formulas
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:77dd14b6994cea530d6441d2609bf8b58c2c0f83b9b99b448fde627e588b3acb
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
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-elemental-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the FSA paper's four dataset-qualification criteria (CHONSP-only formulas with C/H mandatory, monoisotopic mass < 1500 Da, [M+H]+ or [M-H]- precursor, at least three centroid peaks) to raw HMDB, GNPS, and CASMI MS/MS spectra to build the paper's eight test datasets and reproduce Table 1's qualified counts.

## When to use

Use when preparing curated benchmark MS/MS datasets from HMDB, GNPS, or CASMI spectra and you must qualify entries by elemental composition, mass, precursor type, and peak count.

## When NOT to use

- Spectra whose precursor type is anything other than [M+H]+ or [M-H]-, since these are excluded by criterion 3
- Compound libraries containing elements beyond C, H, O, N, S, and P — the paper explicitly has not validated FSA on molecules outside this element set
- Compounds with monoisotopic mass >= 1500 Da, which criterion 2 removes

## Inputs

- Raw HMDB, GNPS, and CASMI MS/MS spectral source files with per-spectrum chemical formula, monoisotopic mass, precursor type, and centroid peak list
- Vendor-format MS files convertible via msconvert (ProteoWizard)

## Outputs

- Eight filtered test datasets (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPID_OT, CASMI_OT)
- Per-dataset tabulation of spectra and distinct metabolite counts before and after selection (Table 1 reproduction)

## How to apply

Parse each raw HMDB/GNPS/CASMI spectrum to extract its chemical formula, monoisotopic mass, precursor type, and centroid peak list, converting vendor files with msconvert from ProteoWizard. Apply the four criteria sequentially: (1) keep only formulas composed exclusively of C, H, O, N, S, and P with both C and H present; (2) keep monoisotopic molecular weight < 1500 Da; (3) keep only [M+H]+ or [M-H]- precursors; (4) require at least three centroid/deisotoped peaks. Count surviving spectra and distinct metabolites per dataset, tabulating them next to raw totals (e.g., HMDB_QTOF drops from 3743/995 to 3029 spectra/821 compounds). Validate per-dataset qualified counts against Table 1 (HMDB 24909 spectra/5630 metabolites; GNPS 5153/1036; CASMI 628/362). These qualified datasets then serve as input to FSA formula ranking, which required no training data and averaged 0.024 s per spectrum.

## Related tools

- **ProteoWizard** (msconvert is used as the free software to convert/prepare the raw MS/MS spectral source files before parsing formulas, masses, precursors, and peak lists for filtering)
- **FSA** (Downstream consumer of the eight qualified datasets; the filter builds the benchmark on which FSA generates and ranks candidate molecular formulas) — https://github.com/kslynn128171/FSA

## Examples

```
msconvert raw_spectra.raw --mzML --outdir mzml/
```

## Evaluation signals

- Qualified per-dataset counts match Table 1: HMDB_QTOF 3029 spectra/821 compounds, HMDB_ITFT 2601/242, HMDB_QFT 1439/221, NPL_MAXIS 3561/2833, NPL_QTOF 1373/1360, NPL_OT 270/239, LIPID_OT 18,084/896, CASMI_OT 333/302
- Aggregated totals match: HMDB 24909 spectra/5630 metabolites, GNPS 5153 spectra/1036 metabolites, CASMI 628 spectra/362 metabolites
- Every retained formula contains only C, H, O, N, S, and P with both C and H present
- Retained masses are all < 1500 Da and precursors are exclusively [M+H]+ or [M-H]-
- Each retained spectrum has >= 3 centroid peaks, and qualified counts are strictly lower than raw counts (e.g., HMDB_QTOF 3743 -> 3029)

## Limitations

- The filter set reflects the paper's scope: formulas restricted to CHONSP, and FSA has not been validated for other elements or for compounds forming various adduct types
- Few chemical rules are implemented to rank or filter out unfeasible formulas; species/adduct-specific rules are not yet incorporated, so filtering alone does not resolve formula ambiguity
- FSA's PubChem snapshot is outdated (109,383,767 compounds vs 112,419,402 as of 2022/11/27), so qualifying downstream candidate formulas may miss newer entries

## Evidence

- [other] Apply criterion 1: keep only spectra whose metabolite formula contains exclusively C, H, O, N, S, and P with both C and H present.: [evidence span withheld — anchor status: not_found]
