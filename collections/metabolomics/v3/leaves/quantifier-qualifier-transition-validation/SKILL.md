---
name: quantifier-qualifier-transition-validation
description: Use when configuring MRM acquisition methods or setting up automRm metabdb.xlsx
  entries, to decide whether each metabolite needs qualifier transitions beyond its
  quantifier for reliable automatic peak reporting.
when_to_use_negative:
- When the acquisition method contains only quantifier transitions with no qualifiers,
  since automRm requires at least one qualifier for optimal performance and cannot
  reach its best peak-reporting quality.
- When maximizing analyte multiplexing is the priority, because the addition of qualifiers
  to the acquisition method reduces the number of analytes that can be detected simultaneously
  in MRM mode.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: automRm
  role: Scores peak candidates with 20 quality scores across quantifier/qualifier
    transitions and applies the peak reporting ML model to decide which peaks are
    reported
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: openxlsx
  role: Reads and writes the metabdb.xlsx transition settings file and the xlsx output
    reports
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:1460b212d9e4178bda2c4f43922b46874a1639d7bf46e42a78270024851cefd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - devtools
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# quantifier-qualifier-transition-validation

## Summary

Validates MRM chromatographic peaks using qualifier transitions in addition to the quantifier transition, as required by automRm's machine-learning peak picking and reporting for LC-QQQ-MS data.

## When to use

Use when configuring MRM acquisition methods or setting up automRm metabdb.xlsx entries, to decide whether each metabolite needs qualifier transitions beyond its quantifier for reliable automatic peak reporting.

## When NOT to use

- When the acquisition method contains only quantifier transitions with no qualifiers, since automRm requires at least one qualifier for optimal performance and cannot reach its best peak-reporting quality.
- When maximizing analyte multiplexing is the priority, because the addition of qualifiers to the acquisition method reduces the number of analytes that can be detected simultaneously in MRM mode.

## Inputs

- metabdb.xlsx metabolite-specific settings file with quantifier/qualifier transitions and expected retention times
- mzML raw MRM data files containing both quantifier and qualifier transitions
- sample metadata from sample.info

## Outputs

- peakinfo.xlsx report with per-metabolite peak quality scores and reporting decisions
- peakoverview.pdf chromatogram plots showing quantifier/qualifier peak agreement
- shiftplots.pdf retention-time-shift plots

## How to apply

For each metabolite, define the quantifier transition plus at least one qualifier transition in the metabolite-specific settings file (metabdb.xlsx), since automRm's 20 quality scores and its ML peak picking/reporting models rely on qualifier information for optimal performance. During process_batch(), peak candidates (default 5 per metabolite per sample) are scored across quantifier and qualifier traces, and the peak reporting ML model filters which peaks are reported. Weigh the added confidence against the cost: adding qualifiers reduces the number of analytes that can be measured simultaneously in MRM mode, so limit qualifiers to metabolites where identification ambiguity is a real risk. Validate the configuration by checking that the generated peakinfo.xlsx report and peakoverview.pdf plots show correctly reported peaks rather than peaks filtered out for qualifier inconsistency.

## Related tools

- **automRm** (Scores peak candidates with 20 quality scores across quantifier/qualifier transitions and applies the peak reporting ML model to decide which peaks are reported) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **openxlsx** (Reads and writes the metabdb.xlsx transition settings file and the xlsx output reports)

## Examples

```
process_batch()  # after configuring quantifier/qualifier transitions in metabdb.xlsx and update_prm.tsv
```

## Evaluation signals

- metabdb.xlsx defines at least one qualifier in addition to the quantifier for each metabolite intended for automatic reporting.
- The peak reporting ML model in process_batch() returns reported peaks for the metabolites with qualifier support, as recorded in peakinfo.xlsx.
- Qualifier-derived quality scores are consistent between the quantifier and qualifier traces for correctly identified peaks.
- Results match the quality of manual peak review on replicate measurements when sufficient relevant training data was used.

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier; without one, peak reporting quality degrades.
- The addition of qualifiers to the acquisition method reduces the number of analytes that can be detected simultaneously in MRM mode.
- With insufficient training on relevant raw data, the ML-based reporting may not reach quality similar to manual peak review.
- Manual peak review may still be needed for excessively dirty samples, very-low-input samples, or new LC methods.

## Evidence

- [discussion] qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] multiplexing cost: "the addition of qualifiers to the acquisition method reduces the number of analytes that can be detected simultaneously in the MRM mode."
- [methods] ml peak reporting: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [methods] end-to-end processing: "process_batch() for preprocessing of batches of raw data"
