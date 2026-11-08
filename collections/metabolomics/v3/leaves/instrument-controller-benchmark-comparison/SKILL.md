---
name: instrument-controller-benchmark-comparison
description: Use when multiple DDA acquisition controllers (e.g., TopN, SmartROI,
  WeightedDEW with specific t0/t1 parameters) must be ranked by how many picked peaks
  they fragment, against a computed optimal, on the same complex sample injections.
when_to_use_negative:
- Input is already a picked-peak/coverage table or fragment spectral quality is the
  target metric rather than coverage of picked peaks
- The acquisition strategy is DIA or the comparison target is MS2 spectral quality
  rather than DDA fragmentation coverage for a single injection
- No picked-peak ground truth (e.g., no mzMine 2 RT/mz bounding boxes) exists for
  the sample, so coverage cannot be defined
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype, implement, optimize,
    and validate DDA controllers in silico before instrument testing
- name: IAPI
  role: Instrument Application Programming Interface used to run the optimized controllers
    on real samples for validation
- name: mzMine 2
  role: Peak picking on full-scan mzML files (SI Table SI-1 parameters) to produce
    the RT/mz bounding boxes that define the coverage ground truth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:f1ef64c69b627e310a310ea207d918295219c4ce37cc32ff2285461fea5c0132
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - ViMMS
  - mzMine 2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# instrument-controller-benchmark-comparison

## Summary

Benchmarks data-dependent acquisition (DDA) controllers — TopN, SmartROI, and WeightedDEW — by computing the coverage of mzMine 2-picked peaks that received at least one MS2 fragmentation event, relative to a theoretically optimal upper bound, using ViMMS for in silico prototyping and IAPI for real instrument injections on serum (QCA) and beer (QCB) extracts.

## When to use

Use when multiple DDA acquisition controllers (e.g., TopN, SmartROI, WeightedDEW with specific t0/t1 parameters) must be ranked by how many picked peaks they fragment, against a computed optimal, on the same complex sample injections.

## When NOT to use

- Input is already a picked-peak/coverage table or fragment spectral quality is the target metric rather than coverage of picked peaks
- The acquisition strategy is DIA or the comparison target is MS2 spectral quality rather than DDA fragmentation coverage for a single injection
- No picked-peak ground truth (e.g., no mzMine 2 RT/mz bounding boxes) exists for the sample, so coverage cannot be defined

## Inputs

- Full-scan mzML files for serum (QCA) and beer (QCB) extracts
- mzML files from instrument injections of each controller (e.g., full-scan, TopN, SmartROI shift 1/2, WeightedDEW t0 = 15 s / t1 = 120 s, optimal reference)
- mzMine 2 picked peaks exported as RT/mz bounding boxes (parameters from SI Table SI-1)
- Controller acquisition parameters to be compared (e.g., WeightedDEW t0/t1)

## Outputs

- Per-controller, per-sample coverage counts (number of picked peaks fragmented by at least one MS2 event)
- Comparison table of controller coverage vs theoretically optimal upper bound (Table 1-style)
- Ranking of DDA controllers, e.g., WeightedDEW as best performer in both serum and beer iterations

## How to apply

First establish a ground truth by loading the full-scan mzML files for the serum (QCA) and beer (QCB) extracts and picking peaks in mzMine 2 with the parameters from Supporting Information Table SI-1, exporting peaks as RT/mz bounding boxes. Then load the mzML files from the instrument injections to compare — here full-scan, TopN, SmartROI shift 1, SmartROI shift 2, WeightedDEW with t0 = 15 s and t1 = 120 s, and the optimal reference — and extract MS2 fragmentation events with precursor m/z and retention times. Match each MS2 event to the picked-peak bounding boxes and define coverage as the number of picked peaks containing at least one fragmentation event. Tabulate coverage per controller per sample alongside the theoretically optimal upper bound (computed taking into account the uneven elution distribution of the ions) so room for improvement is quantified; in this study TopN reached only 656 vs 1542 optimal on serum and 1046 vs 2955 optimal on beer, while WeightedDEW attained the highest coverage in both iterations. Prototype and optimize new controllers in ViMMS before validating on real samples via IAPI, which reduces the traditional need for large numbers of instrument experiments.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype, implement, optimize, and validate DDA controllers in silico before instrument testing)
- **IAPI** (Instrument Application Programming Interface used to run the optimized controllers on real samples for validation)
- **mzMine 2** (Peak picking on full-scan mzML files (SI Table SI-1 parameters) to produce the RT/mz bounding boxes that define the coverage ground truth)

## Evaluation signals

- WeightedDEW (t0 = 15 s, t1 = 120 s) attains the highest coverage of picked peaks among the five controllers in both serum and beer iterations
- Reconstructed coverage counts reproduce the Table 1 values for each controller and sample
- Gap to optimal is consistent with reported values: TopN 656 vs 1542 on serum and 1046 vs 2955 on beer
- Every MS2 fragmentation event matched to exactly the picked peaks whose RT/mz bounding boxes contain its precursor m/z and retention time
- Coverage counts are integers bounded above by the total number of picked peaks and by the optimal upper bound

## Limitations

- DDA typically only fragments a small subset of the ions present, so coverage is inherently incomplete relative to the optimal
- The optimal value is a theoretical upper bound computed for a particular injection, taking into account the uneven elution distribution of the ions — it is not an achievable measurement
- Whether DDA or DIA is the better MS2 acquisition strategy in metabolomics remains inconclusive, so coverage-based controller ranking does not settle that broader question
- Coverage depends on the mzMine 2 peak-picking parameters (SI Table SI-1); different parameters change the ground-truth peak set

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] theoretical results show how much improvement is possible over current DDA strategies: "we first provide theoretical results that show how much improvement is possible over current DDA strategies"
- [intro] prototyped, implemented, optimized, and validated using ViMMS: "Second, we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
- [intro] two new DDA strategies validated on two complex samples: "Finally, we describe two new DDA strategies prototyped in this way and demonstrate, through validation on two complex samples, their improvement over traditional DDA approaches."
