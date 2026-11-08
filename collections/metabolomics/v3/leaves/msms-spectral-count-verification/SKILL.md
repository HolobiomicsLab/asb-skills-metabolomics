---
name: msms-spectral-count-verification
description: Use when deconvoluted MS/MS spectra have been exported to .msp format
  from raw LC-MS/MS data and you must validate that the number of unique spectra per
  ionization mode matches reported/published counts before downstream modeling.
when_to_use_negative:
- Input is already a feature table, annotation matrix, or predicted-fingerprint output
  — counts of MS/MS blocks in .msp format are not applicable at that stage
- Spectra have not been deduplicated within an ionization mode, since duplicated blocks
  will inflate counts and produce spurious mismatches
- The comparison target is annotation coverage (e.g. 638/3,386 and 299/1,901 library
  hits) rather than the total unique-spectrum inventory
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.IPA
  role: Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and
    associated MS/MS scans feeding the count-verification pipeline
- name: IDSL.CSA
  role: Deconvolutes MS/MS spectra, associates fragment spectra with precursor features,
    and exports unique spectra in .msp format whose block counts are verified
- name: IDSL_MINT
  role: Downstream consumer of the verified .msp evaluation set; README documents
    that MSP blocks must include PrecursorMZ row entries, a structural prerequisite
    for valid counting
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:3af1c71d2b383e39428b1d7973e752447c1d15abb183a76186e59df2398db304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL_MINT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# msms-spectral-count-verification

## Summary

Verifies that deconvoluted MS/MS spectra extracted from raw LC-MS/MS data (via IDSL.IPA + IDSL.CSA) match the expected unique-spectrum counts per ionization mode by counting spectrum blocks in NIST .msp files — here 3,386 in ESI+ and 1,901 in ESI− for Metabolomics WorkBench study ST002044.

## When to use

Use when deconvoluted MS/MS spectra have been exported to .msp format from raw LC-MS/MS data and you must validate that the number of unique spectra per ionization mode matches reported/published counts before downstream modeling.

## When NOT to use

- Input is already a feature table, annotation matrix, or predicted-fingerprint output — counts of MS/MS blocks in .msp format are not applicable at that stage
- Spectra have not been deduplicated within an ionization mode, since duplicated blocks will inflate counts and produce spurious mismatches
- The comparison target is annotation coverage (e.g. 638/3,386 and 299/1,901 library hits) rather than the total unique-spectrum inventory

## Inputs

- Raw LC-MS/MS data files for Metabolomics WorkBench study ST002044
- Unique MS/MS spectra exported in NIST .msp format (one file per ionization mode), each block containing PrecursorMZ entries
- Reference published spectrum counts (3,386 ESI+ and 1,901 ESI−)

## Outputs

- Per-mode unique spectrum counts (expected: 3,386 ESI+, 1,901 ESI−) with pass/fail status against reference values
- Validated .msp files usable as the IDSL_MINT model evaluation set

## How to apply

After running IDSL.IPA peak-picking on raw LC-MS/MS data and IDSL.CSA deconvolution separately for ESI+ and ESI− modes, export deduplicated spectra in NIST .msp format (each block must contain a PrecursorMZ row entry). Count the unique MS/MS spectrum blocks in each output .msp file — e.g. by parsing for block-delimiting 'Name:' or 'PrecursorMZ:' entries — and compare per-mode totals against the reference values (3,386 for ESI+, 1,901 for ESI− for ST002044). A mismatch signals a failure in peak detection, precursor-fragment association, or deduplication rather than a data property, so re-examine the IPA/CSA parameters before accepting the count. Because the counts must reproduce exactly for the same raw data and pipeline, this check also serves as the entry criterion for downstream IDSL_MINT fingerprint-prediction workflows that consumed these spectra as the evaluation set.

## Related tools

- **IDSL.IPA** (Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and associated MS/MS scans feeding the count-verification pipeline)
- **IDSL.CSA** (Deconvolutes MS/MS spectra, associates fragment spectra with precursor features, and exports unique spectra in .msp format whose block counts are verified)
- **IDSL_MINT** (Downstream consumer of the verified .msp evaluation set; README documents that MSP blocks must include PrecursorMZ row entries, a structural prerequisite for valid counting) — https://github.com/idslme/IDSL_MINT

## Examples

```
grep -c '^Name:' st002044_pos.msp  # expect 3386; grep -c '^Name:' st002044_neg.msp  # expect 1901
```

## Evaluation signals

- The ESI+ .msp file contains exactly 3,386 unique MS/MS spectrum blocks, matching the reported IDSL.CSA extraction count
- The ESI− .msp file contains exactly 1,901 unique MS/MS spectrum blocks, matching the reported count
- Every counted .msp block includes a required 'PrecursorMZ: ' row entry (a stated MSP-format requirement in the IDSL_MINT README)
- Counts are computed per ionization mode separately after within-mode deduplication; no cross-mode pooling occurs
- A count mismatch is treated as a pipeline failure (IPA/CSA parameter or deduplication error) rather than an acceptable data variation

## Limitations

- The expected counts (3,386 / 1,901) are specific to study ST002044 processed with the IDSL.IPA + IDSL.CSA pipeline; other studies require establishing their own reference counts before verification
- MSP parsing is sensitive to header variations: IDSL_MINT selects the longest content when multiple similar headers appear in a block, and parses both 'comment: ' and 'comments: ' entries, so naive block counting may disagree with tool-internal parsing
- The count check validates inventory size only — it does not verify spectral quality, annotation coverage (which was only 638 and 299 against NIST 20 and LipidBlast), or precursor-fragment assignment correctness

## Evidence

- [results] IDSL.CSA extracted 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− modes for study ST002044: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Raw data processed to extract MS/MS spectra in NIST .msp format as model evaluation set: "First, we processed the raw data for this study to extract MS/MS spectra in the NIST.msp format that can be used as the model evaluation set."
- [readme] MSP blocks must include PrecursorMZ row entries: "MSP blocks must include `PrecursorMZ: ` row entries."
- [results] IDSL.CSA annotated 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes: "IDSL.CSA can annotate 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes by matching the spectra against NIST 20 and LipidBlast"
