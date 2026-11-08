---
name: spectral-library-matching
description: Use when annotating pseudo MS/MS spectra (MGF), full-scan mzML/mzXML
  LC-MS, or imzML imaging data against MS/MS spectral libraries without DDA-MS2 available.
when_to_use_negative:
- Query spectra are genuine DDA MS/MS acquired with the precursor isolated — use standard
  forward library matching instead
- Only molecular formula annotation is needed from MS1 data (e.g. MassCube/METASPACE-style
  target-decoy formula assignment), not spectral-library structural annotation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Python package implementing pseudo MS/MS generation, indexed library search,
    and reverse spectral matching for LC-MS, MSI, and MGF inputs
  repo: https://github.com/Philipbear/ms1_id
- name: IDSL.CSA
  role: Prior proof-of-principle approach for matching pseudo MS/MS (composite) spectra,
    extended here by precursor-tolerant reverse matching with peak scaling
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:b5ca46773ade58ea9107c70b435df73511c05a26e188d160670418873c58e06c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - IDSL.CSA
  - ms1-id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Precursor-tolerant reverse spectral matching of pseudo MS/MS spectra against indexed MS/MS libraries using flash-entropy scoring with mass-dependent peak intensity scaling, enabling structural annotation of MS1-only LC-MS and imaging MS data.

## When to use

Use when annotating pseudo MS/MS spectra (MGF), full-scan mzML/mzXML LC-MS, or imzML imaging data against MS/MS spectral libraries without DDA-MS2 available.

## When NOT to use

- Query spectra are genuine DDA MS/MS acquired with the precursor isolated — use standard forward library matching instead
- Only molecular formula annotation is needed from MS1 data (e.g. MassCube/METASPACE-style target-decoy formula assignment), not spectral-library structural annotation

## Inputs

- pseudo MS/MS spectra in MGF format
- full-scan LC-MS mzML or mzXML files
- imaging MS imzML + ibd files
- indexed MS/MS spectral libraries (e.g. indexed GNPS MS/MS library, July 17 2024)

## Outputs

- annotation TSV/CSV with compound name, precursor m/z, reverse cosine score, matched peak count, spectral usage
- aligned feature table with MS1 and MS2 annotations for LC-MS projects
- dereplicated MSI annotation table (ms1_id_annotations_derep.tsv)

## How to apply

Generate pseudo MS/MS spectra from in-source fragments via correlation analysis of XICs (LC-MS) or ion images (MSI), then match them against an indexed GNPS library with flash-entropy scoring (reverse cosine). Apply mass-dependent peak intensity scaling by raising each fragment m/z ratio α_i to a factor k=10 (gnps_k10.pkl) and use square-root intensity transformation; search both unscaled and k10 libraries simultaneously. Keep the search precursor-tolerant (open): require the reference precursor to appear among the query pseudo MS/MS m/z values, and discard unmatched query peaks when scoring, so extra adduct/multimer/contaminant ions do not penalize the match. Filter with cutoffs — LC-MS: reverse cosine ≥0.7, ≥3 matched peaks, spectral usage ≥0.20; MSI: same score/peaks but spectral usage ≥0.05.

## Related tools

- **ms1-id** (Python package implementing pseudo MS/MS generation, indexed library search, and reverse spectral matching for LC-MS, MSI, and MGF inputs) — https://github.com/Philipbear/ms1_id
- **IDSL.CSA** (Prior proof-of-principle approach for matching pseudo MS/MS (composite) spectra, extended here by precursor-tolerant reverse matching with peak scaling)

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- All reported annotations satisfy cutoffs: reverse cosine ≥0.7, matched peaks ≥3, spectral usage ≥0.20 (LC-MS) or ≥0.05 (MSI)
- Each retained annotation's precursor m/z appears among the m/z values of the query pseudo MS/MS spectrum
- Top-1 hit per unique precursor m/z among all candidates per pseudo MS/MS spectrum
- Ground-truth compounds reach score >0.7 only with k=10 peak scaling applied (validated on 14 known molecules in bile acid/drug standard pools)
- Known annotations recovered: PC(16:0/18:1) [M+H]+ and [M+Na]+ at scores 0.85 and 0.77 in mouse brain MSI

## Limitations

- Low-energy in-source fragments mismatch public reference spectra mostly collected at ≥5 eV collision energy; <1% of MoNA reference spectra were acquired at ≤5 eV, so peak scaling is needed
- Runtime scales with data: ~5–7 min per LC-MS file (increase --mass_detect_int_tol beyond 2e5 Orbitrap / 5e2 QTOF if slower); ~3–20 min per MSI dataset with ≥12 cores
- Isotope peaks (flagged by MassCube) must be excluded from pseudo MS/MS generation or they corrupt correlation clustering
- MS imaging annotation uses the minmz100 library variant with fragments m/z <100 removed

## Evidence

- [discussion] k10 scaling cutoff: [evidence span withheld — anchor status: not_found]
- [results] PC validation: [evidence span withheld — anchor status: not_found]
- [readme] README run: [evidence span withheld — anchor status: external_doc]
- [readme] README scope: [evidence span withheld — anchor status: external_doc]
