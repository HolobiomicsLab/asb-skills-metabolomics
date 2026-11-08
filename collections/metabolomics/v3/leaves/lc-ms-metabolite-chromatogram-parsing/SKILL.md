---
name: lc-ms-metabolite-chromatogram-parsing
description: Use when raw LC-MS MRM data are supplied as mzML files (e.g. from an
  Agilent 1290 Infinity II UHPLC coupled to an Agilent 6495 QQQ-MS) and chromatograms
  per metabolite, per sample must be extracted for preprocessing.
when_to_use_negative:
- Data are already preprocessed peak tables or feature matrices — no mzML chromatogram
  parsing is needed.
- Acquisition method contains no qualifier transition in addition to the quantifier
  — automRm explicitly requires at least one qualifier for optimal performance.
- Input is in a vendor-proprietary format not convertible to mzML; the automRm parser
  depends on mzR for mzML files.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: automRm
  role: R package providing the batch preprocessing pipeline whose first stage parses
    metabolite chromatograms from mzML files via mzR
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Underlying parser used by automRm for reading mzML files into chromatogram
    data structures
- name: process_batch()
  role: automRm entry point that consumes the parsed chromatograms for full batch
    preprocessing
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:e77a61c7850d91d6a687459b57ca0bfea3c65c22e04ab453a1ae2f2a7a4fb739
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - automRm
  - mzR
  - process_batch()
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-ms-metabolite-chromatogram-parsing

## Summary

Parse metabolite chromatograms (quantifier and qualifier traces) from LC-QQQ-MS MRM batches using automRm's mzR-based mzML import, then extract smoothed RT-resolved peak traces per metabolite and sample for downstream peak picking, ML scoring, and RT alignment.

## When to use

Use when raw LC-MS MRM data are supplied as mzML files (e.g. from an Agilent 1290 Infinity II UHPLC coupled to an Agilent 6495 QQQ-MS) and chromatograms per metabolite, per sample must be extracted for preprocessing.

## When NOT to use

- Data are already preprocessed peak tables or feature matrices — no mzML chromatogram parsing is needed.
- Acquisition method contains no qualifier transition in addition to the quantifier — automRm explicitly requires at least one qualifier for optimal performance.
- Input is in a vendor-proprietary format not convertible to mzML; the automRm parser depends on mzR for mzML files.

## Inputs

- mzML files containing quantifier and qualifier chromatograms for LC-MS MRM batches
- Metabolite definition table mapping metabolites to their quantifier/qualifier MRM transitions
- Batch sample annotation (sample names, injection order)

## Outputs

- Per-metabolite, per-sample quantifier and qualifier chromatogram matrices in R
- Local-maxima candidate peak lists ranked by peak height on smoothed quantifier traces
- Preprocessed chromatogram objects passed to automRm ML peak picking and RT alignment stages

## How to apply

Load each batch's mzML files with automRm's mzR-backed parser to obtain the quantifier and qualifier chromatograms for every metabolite in every sample (e.g. from the HILIC 1a demodata set). Peak picking operates by detecting local maxima in a smoothed version of the quantifier trace and ranking them by peak height, so chromatograms must be parsed with sufficient resolution around the expected RT. The parsing step supplies the raw chromatogram matrices that feed automRm's process_batch() preprocessing, ML peak picking, and cross-correlation RT alignment, and is the required input format for train_model().

## Related tools

- **automRm** (R package providing the batch preprocessing pipeline whose first stage parses metabolite chromatograms from mzML files via mzR) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Underlying parser used by automRm for reading mzML files into chromatogram data structures)
- **process_batch()** (automRm entry point that consumes the parsed chromatograms for full batch preprocessing) — https://gitlab.gwdg.de/joerg.buescher/automrm

## Examples

```
library(automRm); process_batch("path/to/mzml_dir", "metab_def.csv", "samples.csv")
```

## Evaluation signals

- Parsed chromatograms must contain both quantifier and qualifier traces for every metabolite-sample pair before peak picking is attempted.
- Local maxima detected on the smoothed quantifier trace must yield a ranked candidate peak list per metabolite, with the highest-height peak matching the known RT for well-behaved analytes.
- On the HILIC 1a demodata set, parsed chromatograms should reproduce the published quantifier/qualifier RT pairs without misalignment between samples.
- The parsed chromatogram structures must be directly consumable by automRm's process_batch() and train_model() without format conversion.
- Peak integration variability among replicate measurements computed downstream should match the low variability reported for automRm on HILIC-MS data if parsing was lossless.

## Limitations

- automRm requires the use of at least one qualifier in addition to the quantifier for optimal performance; parsing batches lacking qualifiers degrades downstream peak picking.
- Retreat to manual peak review is still needed for excessively dirty samples, very-low-input samples, or new LC methods, so parsed chromatograms alone do not guarantee correct reporting.
- With insufficient training on relevant raw data, results can fall below the quality of manual peak review even when parsing is correct.

## Evidence

- [methods] mzR-backed mzML parsing: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [discussion] qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [methods] peak picking basis: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] demodata location: "available online (https://gitlab.gwdg.de/joerg.buescher/demodata)"
- [methods] instrument and method context: "using an Agilent 1290 Infinity II UHPLC in line with an Agilent 6495 QQQ-MS"
