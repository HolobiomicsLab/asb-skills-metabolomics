---
name: in-silico-bond-fragment-generation
description: Use when curating raw spectral library entries (NIST'17, MS-DIAL .msp,
  MSnLib *_ms2.mgf) into fragment-annotated, training-ready spectra for a bond-cleavage-based
  MS/MS prediction model.
when_to_use_negative:
- Spectra already carry validated fragment annotations and only need model training,
  not re-curation
- The target spectra are EI-MS or use precursor/ionization types outside [M+H]+, [M-H]-,
  [M]+, [M]-
- Multi-step fragmentation is required — this skill enumerates single-edge (single-bond)
  cleavages only, which is FIORA's stated biggest limitation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Target model whose training pipeline uses single-edge fragment generation
    and fragment-to-peak matching to curate NIST'17, MS-DIAL, and MSnLib spectra into
    library.csv
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:f73cfb142e0df67391ed3fbd7852041faf433cdb9d2622fdd9522e193d1a7229
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# in-silico-bond-fragment-generation

## Summary

Systematically enumerate single-edge molecular fragments by cleaving each bond of a compound independently (allowing up to four hydrogen losses to distribute charge between charged fragment and neutral loss), then match the generated fragment m/z values against observed MS/MS peaks to build quality-controlled training spectra for FIORA.

## When to use

Use when curating raw spectral library entries (NIST'17, MS-DIAL .msp, MSnLib *_ms2.mgf) into fragment-annotated, training-ready spectra for a bond-cleavage-based MS/MS prediction model.

## When NOT to use

- Spectra already carry validated fragment annotations and only need model training, not re-curation
- The target spectra are EI-MS or use precursor/ionization types outside [M+H]+, [M-H]-, [M]+, [M]-
- Multi-step fragmentation is required — this skill enumerates single-edge (single-bond) cleavages only, which is FIORA's stated biggest limitation

## Inputs

- Raw NIST'17 MS/MS library entries
- MS-DIAL spectral library in .msp format
- MSnLib *_ms2.mgf spectral files (downloaded from Zenodo, e.g. record 16984129)
- Compound structures with SMILES, MOL format, and InChIKey identifiers

## Outputs

- Curated fragment-annotated spectra written to library.csv
- Per-spectrum fragment-peak matches with m/z tolerance and intensity coverage statistics

## How to apply

First parse the raw NIST'17, MS-DIAL, and MSnLib entries, convert nominal collision energies (NCE) to eV, and drop spectra with missing or unclear CE formats; filter for ionization types [M+H]+ and [M-H]- (additionally [M]+ and [M]- for MSnLib), remove compounds with SMILES/MOL/InChIKey inconsistencies, and enforce hard limits of 1000 Da max weight and 100 eV max collision energy. For each retained compound, evaluate every bond cleavage independently, allowing up to four hydrogen losses so charge is distributed between the charged fragment and its neutral loss. Match generated fragments to observed spectral peaks at a 50 ppm m/z tolerance (10 ppm for MSnLib) and record the fraction of total peak intensity covered. Retain a spectrum only if it has at least two peak matches, minimum 50% peak intensity coverage, and a maximum precursor intensity of 90%, applying the soft filters (50% of total peaks or at least five peaks matched, or exceptionally high intensity coverage of 0.8). Rationale: single-edge fragments grounded in the local molecular neighborhood are exactly the supervision signal FIORA uses to learn per-bond cleavage probabilities.

## Related tools

- **FIORA** (Target model whose training pipeline uses single-edge fragment generation and fragment-to-peak matching to curate NIST'17, MS-DIAL, and MSnLib spectra into library.csv) — https://github.com/BAMeScience/fiora

## Examples

```
python resources/data/msnlib/download_msnlib.py && python resources/data/msnlib/preprocess_msnlib.py
```

## Evaluation signals

- Every retained spectrum in library.csv satisfies >=2 fragment-peak matches, >=50% peak intensity coverage, and <=90% precursor intensity
- All entries pass hard filters: compound weight <=1000 Da, collision energy <=100 eV, precursor type in {[M+H]+, [M-H]-} (plus [M]+/[M]- for MSnLib)
- No retained compound has inconsistencies between its SMILES, MOL format, and InChIKey
- library.csv exists, is loadable as a table, and every spectrum has a successfully converted collision energy in eV
- MSnLib fragment matching used the tighter 10 ppm tolerance while NIST'17/MS-DIAL used 50 ppm

## Limitations

- Single-edge fragmentation is the biggest limitation of the approach; multi-step fragmentation is not implemented
- FIORA was trained on only ~10,000 unique compounds, limited by missing experimental metadata, especially collision energies
- MS/MS spectra with missing collision energies or unclear formats must be excluded, shrinking the usable data
- The MSnLib downloader defaults to MSnLib v7 on Zenodo, which differs from the v1.0 record (10.5281/zenodo.11163381) cited in the paper

## Evidence

- [discussion] hard retention filters: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] soft filters: "soft filters required that each spectrum have either 50% of the total peaks or at least five peaks matched by fragments"
- [discussion] hard limits on weight and CE: "These requirements included a maximum weight of 1000 Da and maximum collision energy of 100 eV."
- [discussion] structure consistency filter: "Compounds with inconsistencies between SMILES, MOL format and InChiKeys were filtered out immediately."
- [readme] MSnLib download defaults: "The downloader defaults to MSnLib v7 on Zenodo (`https://zenodo.org/records/16984129`) and accepts both direct file URLs and Zenodo record URLs."
