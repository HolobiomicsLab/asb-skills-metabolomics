---
name: centroid-peak-count-filtering
description: Use when building qualified MS/MS test datasets from raw HMDB/GNPS/CASMI
  spectra and you must enforce element-composition, mass (<1500 Da), precursor-type,
  and minimum-centroid-peak (>=3) inclusion rules before formula ranking.
when_to_use_negative:
- The compounds of interest contain elements beyond C, H, O, N, S, P or lack C/H —
  criterion 1 excludes them by design, and FSA has not been validated outside CHONSP
- The precursor is an adduct other than [M+H]+ or [M-H]- (e.g. [M+Na]+, in-source
  fragments) — criterion 3 removes such spectra, so this filter would silently discard
  your data
- You already hold a qualified, counted dataset and only need formula ranking, not
  dataset construction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: msconvert converts vendor raw MS/MS files to open formats and produces the
    centroided peak lists that the >=3-peak and other qualification criteria are applied
    to
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:a2d7071232784ff70119a161e2a2ba0c0e87647e94f2b7dd0df43831f30f79e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  tools:
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Centroid-peak-count filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the paper's four dataset-qualification criteria — CHONSP-only formulas with C and H mandatory, monoisotopic mass < 1500 Da, [M+H]+ or [M-H]- precursor, and at least three centroid/deisotoped peaks — to raw HMDB, GNPS, and CASMI MS/MS spectra, yielding the eight qualified test datasets and reproducing Table 1's before/after counts.

## When to use

Use when building qualified MS/MS test datasets from raw HMDB/GNPS/CASMI spectra and you must enforce element-composition, mass (<1500 Da), precursor-type, and minimum-centroid-peak (>=3) inclusion rules before formula ranking.

## When NOT to use

- The compounds of interest contain elements beyond C, H, O, N, S, P or lack C/H — criterion 1 excludes them by design, and FSA has not been validated outside CHONSP
- The precursor is an adduct other than [M+H]+ or [M-H]- (e.g. [M+Na]+, in-source fragments) — criterion 3 removes such spectra, so this filter would silently discard your data
- You already hold a qualified, counted dataset and only need formula ranking, not dataset construction

## Inputs

- Raw HMDB, GNPS, and CASMI MS/MS spectral source files (vendor or converted mzML)
- Per-spectrum chemical formula, monoisotopic mass, precursor type ([M+H]+ / [M-H]-), and centroid/deisotoped peak list

## Outputs

- Eight qualified test datasets (HMDB_QTOF, HMDB_ITFT, HMDB_QFT, NPL_MAXIS, NPL_QTOF, NPL_OT, LIPID_OT, CASMI_OT)
- Per-dataset table of raw vs. qualified spectrum and compound counts reproducing Table 1 (e.g. HMDB 24909 spectra / 5630 metabolites; GNPS 5153 / 1036; CASMI 628 / 362)

## How to apply

Convert vendor raw files to open format (e.g. mzML) with ProteoWizard's msconvert, then parse each spectrum's chemical formula, monoisotopic mass, precursor type, and centroid peak list. Apply the four criteria sequentially: (1) retain only formulas composed exclusively of C, H, O, N, S, P with both C and H present; (2) retain monoisotopic molecular weight < 1500 Da; (3) retain only [M+H]+ or [M-H]- precursor types (use the fiehnlab adduct calculator to confirm adduct masses); (4) retain spectra with at least three centroid/deisotoped peaks. Count surviving spectra and distinct metabolites per dataset and tabulate alongside raw totals. The rationale is that these criteria define the qualification basis on which FSA's formula-ranking performance (85.28% top-1 accuracy over 30,690 spectra) was measured.

## Related tools

- **ProteoWizard** (msconvert converts vendor raw MS/MS files to open formats and produces the centroided peak lists that the >=3-peak and other qualification criteria are applied to)

## Examples

```
msconvert sample.raw --mzML --filter "peakPicking true 1" --outdir mzml/
```

## Evaluation signals

- Qualified per-dataset counts match Table 1: HMDB_QTOF 3029 spectra (821 compounds) from 3743 (995); HMDB_ITFT 2601 (242); HMDB_QFT 1439 (221); NPL_MAXIS 3561 (2833); NPL_QTOF 1373 (1360); NPL_OT 270 (239); LIPID_OT 18,084 (896); CASMI_OT 333 (302)
- Dataset totals equal HMDB 24909 spectra / 5630 metabolites, GNPS 5153 spectra / 1036 metabolites, CASMI 628 spectra / 362 metabolites
- No retained spectrum has a formula containing elements outside {C, H, O, N, S, P} or missing C or H
- All retained spectra have monoisotopic mass < 1500 Da, precursor [M+H]+ or [M-H]-, and >= 3 centroid peaks
- Retention is applied cumulatively (criteria 1-4 in order), so each dataset's qualified count is <= its raw count

## Limitations

- Element scope restricted to CHONSP; spectra from metabolites with halogens, Si, metals, etc. are excluded and FSA has only been validated on C, H, O, N, S, P compounds
- Hard cutoffs (< 1500 Da, >= 3 peaks, only [M+H]+/[M-H]-) exclude larger metabolites, low-peak spectra, and alternative adducts
- Few chemical rules are implemented downstream to rank or filter unfeasible formulas, and species/adduct-specific rules are not incorporated

## Evidence

- [other] Apply criterion 4: keep only spectra containing at least three centroid/deisotoped peaks.: [evidence span withheld — anchor status: not_found]
- [other] Keep only spectra whose metabolite formula contains exclusively C, H, O, N, S, and P with both C and H present.: [evidence span withheld — anchor status: not_found]
- [intro] free software such as msconvert in ProteoWizard: "free software such as msconvert in ProteoWizard (Chambers et al., 2012)"
- [abstract] FSA does not require training data, not rely on MS/MS databases, can be applied to a wide mass range: "FSA does not require training data, not rely on MS/MS databases, can be applied to a wide mass range, and can be quickly expanded with more chemical elements and formulas"
