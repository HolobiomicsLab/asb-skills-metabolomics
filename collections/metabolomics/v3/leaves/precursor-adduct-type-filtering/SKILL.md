---
name: precursor-adduct-type-filtering
description: Use when building benchmark MS/MS datasets from raw HMDB, GNPS, or CASMI
  spectra and you must restrict entries to protonated or deprotonated precursor types
  before counting qualified spectra and compounds.
when_to_use_negative:
- Benchmarking adduct-aware tools that explicitly handle precursor species beyond
  [M+H]+ and [M-H]-, where filtering would remove the compounds under test
- Analyzing spectra already qualified (Table 1 counts already matched), where re-filtering
  would double-apply criteria and skew benchmark denominators
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ProteoWizard
  role: msconvert is used to obtain spectra (chemical formula, monoisotopic mass,
    precursor type, centroid peak list) from the raw HMDB/GNPS/CASMI source files
    prior to applying the precursor-type filter
  repo: https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:2442abfd2cb6baa917fe551c8075d5387099faeb18dc86e7d47f22292247db17
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# precursor-adduct-type-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A dataset-qualification filter from the FSA evaluation that restricts MS/MS spectra to the precursor types [M+H]+ and [M-H]-, one of the four criteria (alongside CHONSP composition, mass < 1500 Da, and >= 3 peaks) used to build the eight HMDB/GNPS/CASMI test datasets.

## When to use

Use when building benchmark MS/MS datasets from raw HMDB, GNPS, or CASMI spectra and you must restrict entries to protonated or deprotonated precursor types before counting qualified spectra and compounds.

## When NOT to use

- Benchmarking adduct-aware tools that explicitly handle precursor species beyond [M+H]+ and [M-H]-, where filtering would remove the compounds under test
- Analyzing spectra already qualified (Table 1 counts already matched), where re-filtering would double-apply criteria and skew benchmark denominators

## Inputs

- Raw MS/MS spectra from HMDB, GNPS, and CASMI source files with per-spectrum chemical formula, monoisotopic mass, precursor type, and centroid peak list
- Vendor-format or open-format spectral files convertible with ProteoWizard msconvert

## Outputs

- Filtered spectral subsets containing only [M+H]+ and [M-H]- precursor spectra
- Per-dataset counts of qualified spectra and distinct metabolites before and after adduct filtering (reproducing Table 1's selection rows)

## How to apply

Parse each raw spectrum's precursor type alongside its chemical formula, monoisotopic mass, and centroid peak list (e.g., after converting vendor files with ProteoWizard msconvert). Keep only spectra whose precursor type is exactly [M+H]+ or [M-H]-, discarding all other adduct species such as ammonium, sodium, or chloride adducts and neutral-loss precursors. Apply this criterion in combination with the other three selection rules: formula restricted to C, H, O, N, S, P with both C and H mandatory; monoisotopic molecular weight < 1500 Da; and at least three centroid/deisotoped peaks. Then count surviving spectra and distinct metabolites per dataset and compare against the paper's Table 1 before/after selection counts to confirm the filter reproduces the expected qualified totals. The rationale is that FSA's formula subset scoring is validated only on protonation/deprotonation adduct chemistry, so heterogeneous precursor types would introduce unmodeled adduct-specific fragment relationships.

## Related tools

- **ProteoWizard** (msconvert is used to obtain spectra (chemical formula, monoisotopic mass, precursor type, centroid peak list) from the raw HMDB/GNPS/CASMI source files prior to applying the precursor-type filter) — https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/

## Examples

```
msconvert raw.mzML --mzML --filter "peakPicking" && python filter_precursors.py --input raw.mzML --precursors '[M+H]+,[M-H]-' --min-peaks 3 --max-mass 1500
```

## Evaluation signals

- Per-dataset qualified counts match Table 1: e.g., HMDB_QTOF 3029 spectra (821 compounds) from 3743 (995), CASMI_OT 333 (302) from totals
- Across all eight datasets the qualified totals equal the reported HMDB 24909 spectra / 5630 metabolites, GNPS 5153 spectra / 1036 metabolites, CASMI 628 spectra / 362 metabolites
- Every retained spectrum's precursor annotation is exactly [M+H]+ or [M-H]-; no sodium/ammonium/chloride adducts or neutral-loss precursors survive
- Retained spectra also satisfy the co-applied criteria: CHONSP-only formulas with C and H present, monoisotopic mass < 1500 Da, and >= 3 centroid/deisotoped peaks

## Limitations

- Restricting to [M+H]+/[M-H]- excludes compound species and adduct types; the paper notes few species/adduct-specific chemical rules are implemented and that FSA 'has not yet been validated on compounds forming various adduct types'
- Criterion 4 requires at least three centroid/deisotoped peaks, so very sparse spectra are lost even when the precursor type is qualified
- The formula criterion (CHONSP only with C/H mandatory) runs alongside adduct filtering, so compounds with halogens or metals are excluded regardless of precursor type

## Evidence

- [methods] Apply criterion 3: keep only spectra with precursor type [M+H]+ or [M-H]-.: [evidence span withheld — anchor status: not_found]
- [methods] keep only spectra whose metabolite formula contains exclusively C, H, O, N, S, and P with both C and H present: [evidence span withheld — anchor status: not_found]
