---
name: retention-time-alignment
description: Use when processing a batch of MRM mzML files (e.g. HILIC 1a/1b, MTBLS429,
  MTBLS897) whose metabolite peaks shift in RT between samples, and peak boundaries
  must be aligned before peak picking and ML-based peak reporting.
when_to_use_negative:
- The data are not MRM/LC-QQQ-MS chromatograms with known expected RTs (automRm needs
  quantifier/qualifier transitions and expected RTs per analyte in metabdb.xlsx).
- Only a single sample is available and no reference sample exists for cross-correlation-based
  shift estimation.
- The LC method is brand new or the samples are excessively dirty/very-low-input,
  where the authors state they retreat to manual peak review instead of automatic
  processing.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: automRm
  role: 'R package providing the RT alignment logic inside process_batch(): cross-correlation
    of quantifier traces, quadratic RT-shift fitting, prototype chromatogram summing,
    and shiftplots.pdf output'
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses the mzML raw MRM files into chromatogram traces that feed the cross-correlation
    alignment step
- name: caret
  role: Machine-learning backend for the peak-picking/peak-reporting models that consume
    the alignment-corrected chromatograms
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:84cbdfff603c697a0da987c923b3686e9230533700438b6e79ce17926666094d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - devtools
  - mzR
  - openxlsx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# retention-time-alignment

## Summary

Corrects per-metabolite retention-time (RT) drift across an MRM LC-QQQ-MS batch by cross-correlating each sample's chromatograms against a reference sample, fitting the shifts as a quadratic function of RT, and summing RT-shifted chromatograms into a prototype for robust peak detection. This is the alignment stage inside automRm's process_batch() pipeline.

## When to use

Use when processing a batch of MRM mzML files (e.g. HILIC 1a/1b, MTBLS429, MTBLS897) whose metabolite peaks shift in RT between samples, and peak boundaries must be aligned before peak picking and ML-based peak reporting.

## When NOT to use

- The data are not MRM/LC-QQQ-MS chromatograms with known expected RTs (automRm needs quantifier/qualifier transitions and expected RTs per analyte in metabdb.xlsx).
- Only a single sample is available and no reference sample exists for cross-correlation-based shift estimation.
- The LC method is brand new or the samples are excessively dirty/very-low-input, where the authors state they retreat to manual peak review instead of automatic processing.

## Inputs

- mzML raw MRM data files (parsed via mzR) for a batch of samples
- metabdb.xlsx metabolite database with quantifier/qualifier MRM transitions and expected retention times per analyte
- update_prm.tsv parameter file pointing to raw data, pretrained ML models, and metabdb.xlsx
- a reference sample against which cross-correlation is performed

## Outputs

- Per-metabolite, per-sample RT shifts fitted as a quadratic function of RT
- Prototype chromatogram per metabolite (sum of RT-shifted chromatograms) with peak boundaries propagated back to each sample
- shiftplots.pdf retention-time-shift diagnostic plots
- Aligned peak intensities written to peakinfo.xlsx (with peakoverview.pdf chromatogram plots)

## How to apply

During process_batch(), automRm first detects local maxima in smoothed quantifier traces (up to 5 candidate peaks per metabolite per sample), then calculates RT shifts separately for each metabolite in each sample by simultaneous cross-correlation against a reference sample. The RT shifts of all metabolites in a sample are fitted as a quadratic function of RT, so the sample-wide drift is captured as a smooth curve rather than per-peak offsets. The RT-shifted chromatograms of a metabolite across all samples are then summed into a prototype chromatogram, which makes peak detection robust against sample-to-sample drift, and peak boundaries defined on the prototype are propagated back to each individual sample. Alignment relies on metabolite-specific expected RTs from the metabdb.xlsx database and on having quantifier plus at least one qualifier transition; misalignment should be inspected in the generated shiftplots.pdf.

## Related tools

- **automRm** (R package providing the RT alignment logic inside process_batch(): cross-correlation of quantifier traces, quadratic RT-shift fitting, prototype chromatogram summing, and shiftplots.pdf output) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses the mzML raw MRM files into chromatogram traces that feed the cross-correlation alignment step)
- **caret** (Machine-learning backend for the peak-picking/peak-reporting models that consume the alignment-corrected chromatograms)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- shiftplots.pdf shows smooth, sample-consistent RT-shift curves consistent with the quadratic fit of RT shifts across the retention-time range.
- Peak boundaries defined on the summed prototype chromatogram, when propagated back to each sample, overlay the correct chromatographic peaks in peakoverview.pdf.
- Downstream Pearson correlation between automRm peak areas and originally reported intensities exceeds 0.95 for the majority of metabolites (paper: 22/34 in MTBLS429, 20/30 in MTBLS897).
- Variation in peak integration among replicate measurements after alignment is lower than with alternative software, per the paper's benchmark on the HILIC-MS data set.
- No metabolite is dropped solely due to RT misalignment in samples where the peak is visibly present in the raw chromatogram.

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier, which reduces the number of analytes detectable simultaneously in MRM mode.
- Alignment quality depends on sufficient, relevant training data; with sufficient training on relevant raw data, results reach a quality similar to manual peak review.
- Retreat to manual peak review is needed for special cases such as excessively dirty samples, very-low-input samples, or new LC methods.

## Evidence

- [methods] calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [methods] RT shifts of all metabolites in a sample are fitted as a quadratic function of RT: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] sum the RT-shifted chromatograms of a metabolite across all samples: "sum the RT-shifted chromatograms of a metabolite across all samples"
- [discussion] With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review.: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
