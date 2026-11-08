---
name: ppm-tolerance-peak-matching
description: Use when curating raw MS/MS library spectra (NIST'17, MS-DIAL, MSnLib
  .mgf/.msp entries) into quality-controlled training data, where generated fragment
  candidates must be matched to observed peaks before retention filtering.
when_to_use_negative:
- When the goal is predicting spectra for new compounds with FIORA itself rather than
  curating reference libraries for training
- When input spectra lack collision-energy annotations, since spectra with missing
  or unclear collision energies are excluded outright
- When compounds have inconsistencies between SMILES, MOL format, and InChIKeys, as
  these are filtered out immediately rather than matched
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Provides the single-edge fragmentation model whose generated fragments are
    matched to observed peaks, and the library curation/training pipeline (fiora-train)
    consuming the curated library.csv
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:4022a696ab2939c8f87c672a24131d54cbeadefc15183704c88eda61f37ff4af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ppm-tolerance-peak-matching

## Summary

Quality-control step from the FIORA spectral library curation pipeline that matches computationally generated single-edge fragments to observed spectral peaks at fixed m/z tolerances (50 ppm, or 10 ppm for MSnLib) and applies match/intensity-coverage retention filters to yield training-ready MS/MS spectra.

## When to use

Use when curating raw MS/MS library spectra (NIST'17, MS-DIAL, MSnLib .mgf/.msp entries) into quality-controlled training data, where generated fragment candidates must be matched to observed peaks before retention filtering.

## When NOT to use

- When the goal is predicting spectra for new compounds with FIORA itself rather than curating reference libraries for training
- When input spectra lack collision-energy annotations, since spectra with missing or unclear collision energies are excluded outright
- When compounds have inconsistencies between SMILES, MOL format, and InChIKeys, as these are filtered out immediately rather than matched

## Inputs

- Raw MS/MS spectral library entries from NIST'17, MS-DIAL (.msp), and MSnLib (*_ms2.mgf files from Zenodo)
- Compounds with consistent SMILES / MOL format / InChIKey annotations
- Nominal collision energies (NCE) convertible to eV

## Outputs

- Curated spectral library table (library.csv) containing only spectra passing the >=2 peak-match, >=50% intensity-coverage, and <=90% precursor-intensity filters
- Per-spectrum fragment-to-peak match records with intensity coverage values

## How to apply

First parse the raw NIST'17, MS-DIAL, and MSnLib library entries, convert nominal collision energies (NCE) to eV, and drop spectra with missing or unclearly formatted collision energies; restrict to ionization types [M+H]+ and [M-H]- (plus [M]+ and [M]- for MSnLib) and enforce hard limits of maximum compound weight 1000 Da and maximum collision energy 100 eV. For each retained compound, generate single-edge fragments by evaluating each bond cleavage independently, allowing up to four hydrogen losses to distribute charge between the charged fragment and the neutral loss. Match generated fragments to observed peaks at a 50 ppm m/z tolerance (10 ppm for MSnLib) and record intensity coverage, then retain only spectra with at least two peak matches, minimum peak intensity coverage of 50%, and maximum precursor intensity of 90%. Additionally apply the soft filters: either 50% of total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8. The rationale is that these thresholds guarantee every training spectrum has sufficient explained fragment signal while excluding spectra dominated by an unfragmented precursor.

## Related tools

- **FIORA** (Provides the single-edge fragmentation model whose generated fragments are matched to observed peaks, and the library curation/training pipeline (fiora-train) consuming the curated library.csv) — https://github.com/BAMeScience/fiora

## Examples

```
python resources/data/msnlib/download_msnlib.py && python resources/data/msnlib/preprocess_msnlib.py
```

## Evaluation signals

- Every retained spectrum has at least two fragment-to-peak matches at the tolerance (50 ppm, or 10 ppm for MSnLib)
- All retained entries show minimum peak intensity coverage of 50% and maximum precursor intensity of 90%
- No entry exceeds 1000 Da compound weight or 100 eV collision energy, and only [M+H]+/[M-H]- (plus [M]+/[M]- for MSnLib) precursor types remain
- library.csv exists, loads as a table, and fragment counts per spectrum satisfy the soft filter (>=50% of total peaks, >=5 peaks matched, or intensity coverage >= 0.8)
- Compounds with SMILES/MOL/InChIKey inconsistencies are absent from the output

## Limitations

- MSnLib uses a stricter 10 ppm matching tolerance than the 50 ppm applied to NIST'17 and MS-DIAL, so tolerances must be set per source library
- Single-step fragmentation only: generated fragments allow up to four hydrogen losses but no multi-step (multi-cleavage) fragments, limiting matchable peak space
- The hard 1000 Da weight and 100 eV collision-energy caps exclude larger compounds and high-energy spectra from the curated library

## Evidence

- [discussion] Hard filters: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] Soft filters: "soft filters required that each spectrum have either 50% of the total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8."
- [discussion] Hard limits: "These requirements included a maximum weight of 1000 Da and maximum collision energy of 100 eV."
- [readme] Downloader pattern: "For Zenodo record URLs it downloads all files matching `*_ms2.mgf` by default."
