---
name: instrument-performance-benchmarking
description: Use when you have paired full-scan and DDA-controller mzML files (e.g.
  serum QCA / beer QCB injections) and need to quantify how many picked peaks were
  actually fragmented versus the theoretical optimum for those injections.
when_to_use_negative:
- Input is already a feature table or quantified peak intensity matrix with no scan-level
  mzML to match MS2 events to peaks
- The goal is evaluating DIA rather than DDA coverage — the article frames this benchmark
  for DDA controllers and notes DDA vs DIA comparisons remain inconclusive
- You need to compare acquisition strategies on synthetic data; ViMMS in silico simulation
  is the appropriate route instead of real-instrument benchmarking
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Peak-picking of full-scan mzML using SI Table SI-1 parameters; exports picked
    peaks as RT/m/z bounding boxes
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and benchmark DDA
    controller strategies alongside the real TopN injections
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:81ba8b3e16e3e5e89b5f2147ada8505676ca5049af1fa3ccaf1e40178008ae90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - Peakonly
  - ViMMS
  - XCMS
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# instrument-performance-benchmarking

## Summary

Benchmarks the coverage and efficiency of a standard TopN DDA controller against the theoretically optimal fragmentation performance for real injections, using picked peak bounding boxes matched to observed MS2 events on mzML data.

## When to use

Use when you have paired full-scan and DDA-controller mzML files (e.g. serum QCA / beer QCB injections) and need to quantify how many picked peaks were actually fragmented versus the theoretical optimum for those injections.

## When NOT to use

- Input is already a feature table or quantified peak intensity matrix with no scan-level mzML to match MS2 events to peaks
- The goal is evaluating DIA rather than DDA coverage — the article frames this benchmark for DDA controllers and notes DDA vs DIA comparisons remain inconclusive
- You need to compare acquisition strategies on synthetic data; ViMMS in silico simulation is the appropriate route instead of real-instrument benchmarking

## Inputs

- Full-scan mzML files for the serum (QCA) and beer (QCB) extracts
- mzML files from the TopN DDA controller injections of the same samples
- Picked peak lists exported from MZmine2 as RT/m/z bounding boxes (parameters per SI Table SI-1)

## Outputs

- results_table.tsv with per-sample TopN coverage, efficiency, and MS1/MS2 scan counts
- Per-sample theoretical optimum peak counts (e.g. 1542 serum, 2955 beer) for comparison

## How to apply

Pick peaks on the full-scan mzML files of the serum (QCA) and beer (QCB) extracts with MZmine2 using the SI Table SI-1 parameters, and export each peak as a bounding box defined by min/max RT (first to last MS1 scan of the chromatographic peak) and min/max m/z. Read the mzML files from the TopN controller injections and extract every MS2 fragmentation event, then match each event to the peak bounding boxes it falls into. Compute coverage as the number of picked peaks containing at least one fragmentation event, and efficiency as the ratio of picked peaks fragmented to the number of MS2 scans; additionally record MS1 and MS2 scan counts. The rationale, grounded in the article, is that TopN DDA typically fragments only a small subset of ions, so benchmarking against a theoretically optimal performance computed for the same injection (accounting for uneven ion elution) quantifies the headroom for improved DDA prioritization strategies.

## Related tools

- **MZmine 2** (Peak-picking of full-scan mzML using SI Table SI-1 parameters; exports picked peaks as RT/m/z bounding boxes)
- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and benchmark DDA controller strategies alongside the real TopN injections)

## Evaluation signals

- Serum (QCA) TopN coverage should match Table 1: 656 fragmented picked peaks observed vs 1542 theoretically optimal
- Beer (QCB) TopN coverage should match Table 1: 1046 fragmented picked peaks observed vs 2955 theoretically optimal
- MS1 and MS2 scan counts per sample should match the values reported in Table 2
- Efficiency values should be reproducible as picked-peaks-fragmented divided by number of MS2 scans for each injection
- Every reported fragmented peak count must be a subset of the total picked peak count exported from MZmine2 bounding boxes

## Limitations

- The paper notes DDA typically only fragments a small subset of ions, so observed coverage will always fall well below the theoretical optimum even for well-tuned controllers
- Coverage depends on the peak-picking parameters (SI Table SI-1); different pickers (MZmine 2, XCMS, Peakonly) may yield different bounding boxes and thus different coverage estimates

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] we first provide theoretical results that show how much improvement is possible over current DDA strategies: "we first provide theoretical results that show how much improvement is possible over current DDA strategies"
- [intro] Here, we address the problem of improving DDA coverage for a single injection: "Here, we address the problem of improving DDA coverage for a single injection"
- [intro] We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
