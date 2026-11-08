---
name: coeluting-isobaric-lipid-feature-analysis
description: Use when an LC-MS/MS lipidomics peak table contains chromatographic features
  with multiple isobaric coeluting lipid spectral matches and you must quantify feature
  purity before assigning a confident identification.
when_to_use_negative:
- Features are already resolved and only statistical QC (e.g. QC-sample %RSD thresholding)
  is needed — Degreaser's automated threshold filtering alone suffices.
- The input is a single-purity, fully validated MS2 spectrum with no isobaric coelution,
  where purity estimation adds no information.
- Working outside lipidomics on proteomics or metabolomics feature tables, where lipid-specific
  fragmentation templates and purity logic do not apply.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: LipiDex 2
  role: Implements the rewritten spectral matching and peak annotation pipeline that
    measures spectral purity and estimates chromatographic feature purity for coeluting
    isobaric lipids
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 1
  role: Original tool that introduced the spectral-purity-to-feature-purity estimation
    for coeluting isobaric lipid species
  repo: https://github.com/coongroup/LipiDex
- name: Degreaser QC module
  role: Consumes purity-validated spectral matches and applies automated threshold
    filtering plus optional manual validation on the resulting lipid features
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: Spectrum Searcher
  role: Produces the MS2/MS3+ spectral matches whose purity is used to estimate feature
    purity
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:95b73f7bd071bcd270b210589844f8a3f24aed0c34ca3fc8812eda257ac43e70
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Degreaser QC module
  - LipiDex 1
  - LipiDex 2
  - Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# coeluting-isobaric-lipid-feature-analysis

## Summary

Handles coeluting isobaric lipid species in LC-MS lipidomics by measuring spectral purity of lipid spectral matches and using it to estimate the purity of the associated chromatographic feature, so that peak identifications contaminated by overlapping isobars can be downgraded or flagged. Central to LipiDex/LipiDex 2's spectral matching and peak-annotation logic.

## When to use

Use when an LC-MS/MS lipidomics peak table contains chromatographic features with multiple isobaric coeluting lipid spectral matches and you must quantify feature purity before assigning a confident identification.

## When NOT to use

- Features are already resolved and only statistical QC (e.g. QC-sample %RSD thresholding) is needed — Degreaser's automated threshold filtering alone suffices.
- The input is a single-purity, fully validated MS2 spectrum with no isobaric coelution, where purity estimation adds no information.
- Working outside lipidomics on proteomics or metabolomics feature tables, where lipid-specific fragmentation templates and purity logic do not apply.

## Inputs

- Chromatographic peak table (from Compound Discoverer .cdResult or .csv)
- MS2 and MS3+ spectral data (.mgf or .mzXML for LipiDex 1; Thermo .raw supported in LipiDex 2)
- In silico lipid spectral library from the LipiDex Library Generator
- Spectral match results from the Spectrum Searcher module

## Outputs

- Per-feature purity estimates reflecting coeluting isobaric contamination
- Annotated peak table with lipid identifications at structural or sum-composition level
- Filtered feature list with spurious/redundant features removed for downstream QC (Degreaser)

## How to apply

During the Spectrum Searcher/peak identification stage, each MS2 or MS3+ spectral match is scored for purity: the fraction of signal in the matched spectrum attributable to the identified lipid rather than a coeluting isobar. LipiDex propagates this spectral-match purity to the associated chromatographic feature as an estimate of feature purity, since isobaric coelution degrades both. In LipiDex 2 the Degreaser QC module additionally requires spectral match validation for all matches in the lipid feature, so a feature whose component matches show low purity or unvalidated fragments is filtered rather than accepted. When evidence for a specific structural detail is insufficient (e.g. the required plasmenyl fragment is absent), the identification is downgraded (to sum composition or plasmanyl) instead of being reported at a false structural level. Outputs feed Degreaser's automated threshold filtering, where features with >30% RSD in QC samples are also flagged for removal, leaving a cleaner annotated peak table.

## Related tools

- **LipiDex 2** (Implements the rewritten spectral matching and peak annotation pipeline that measures spectral purity and estimates chromatographic feature purity for coeluting isobaric lipids) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 1** (Original tool that introduced the spectral-purity-to-feature-purity estimation for coeluting isobaric lipid species) — https://github.com/coongroup/LipiDex
- **Degreaser QC module** (Consumes purity-validated spectral matches and applies automated threshold filtering plus optional manual validation on the resulting lipid features) — https://github.com/coongroup/LipiDex-2/releases/latest
- **Spectrum Searcher** (Produces the MS2/MS3+ spectral matches whose purity is used to estimate feature purity) — https://github.com/coongroup/LipiDex-2/releases/latest

## Examples

```
Download LipiDex2.exe from https://github.com/coongroup/LipiDex-2/releases/latest, extract, run LipiDex2.exe; feed .cdResult peak table + .raw spectra through Spectrum Searcher → Peak Finder → Degreaser.
```

## Evaluation signals

- Every retained lipid feature has spectral match validation performed for all matches in the lipid feature.
- Identification level matches available evidence: absence of a required fragment downgrades the call to sum composition, and insufficient plasmenyl evidence downgrades to plasmanyl rather than plasmenyl.
- Features carrying >30% RSD in QC samples are flagged for removal in the filtered output.
- Filtered results show removal of adducts, dimers, and other spurious features relative to the raw Peak Finder output.
- Pipeline continuity holds: .csv results files pass cleanly between LipiDex 2 modules (Library Generator → Spectrum Searcher → Peak Finder → Degreaser).

## Limitations

- Purity estimation is indirect: it infers chromatographic feature purity from spectral match purity, so coeluting isobars with near-identical fragmentation patterns may be underestimated in contamination.
- LipiDex 1 is no longer supported and accepts only .mgf/.mzXML plus peak tables from Compound Discoverer or mzMine 2; users needing Thermo .raw input must move to LipiDex 2.
- Peak Finder in LipiDex 2 requires Compound Discoverer version 3.3 .cdResult files as input for the peak table, limiting upstream flexibility.
- LipiDex 2's interactive Wiki documentation was work-in-progress (WiP) at the time of writing, which may complicate onboarding.
- Downgrading logic is conservative: insufficient evidence for ether-linkage or positional detail yields a sum-composition-level call rather than an attempted full assignment.

## Evidence

- [intro] LipiDex 1 aimed to account for coeluting isobaric lipid species by measuring spectral purity and using it to estimate chromatographic feature purity: "LipiDex aimed to account for coeluting isobaric lipid species by measuring the purity of lipid spectral matches2, then use the spectral purity to estimate the purity of the associated"
- [readme] Degreaser QC steps: "Degreaser Quality Control module allows for automated validation of lipid identifications and features using 6 pieces of evidence:
  - Effective carbon number versus retention time modeling (ECN vs."
- [methods] Spectral match validation requirement: "Spectral match validation for all matches in the lipid feature"
- [results] RSD filtering threshold: "For example, features with >30% RSD in QC samples will be flagged for removal."
