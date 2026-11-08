---
name: peak-picking-from-fullscan-mzml
description: Reference for turning untargeted metabolomics LC-MS/MS full-scan mzML
  data (e.g. serum or beer extract injections) into per-peak bounding boxes of min/max
  RT and m/z to define which ions any DDA strategy could fragment.
when_to_use_negative:
- Input is already a peak table or feature list — this skill operates on raw full-scan
  mzML, not on pre-computed features.
- The goal is peak picking on MS2 fragmentation spectra rather than the MS1 full-scan
  survey data.
- The instrument acquisition is DIA rather than DDA, where the TopN-driven MS2-scan-slot
  bounding-box semantics do not apply.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: performs the actual peak picking on full-scan mzML files and exports bounding
    boxes of min/max RT and m/z per picked peak
- name: ViMMS
  role: consumes the picked-peak bounding boxes to simulate the virtual metabolomics
    mass spectrometer and evaluate DDA strategies in silico
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:2eeecc72bc045289353d501846207563f5ba44b3c4108bbb852eed193af19d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# peak-picking-from-fullscan-mzml

## Summary

Converts raw full-scan LC-MS1 mzML files into per-peak bounding boxes of m/z and retention-time limits using MZmine 2 peak picking, providing the fragmentable-ion universe that ViMMS and the maximum-matching coverage analyses operate on.

## When to use

Reference for turning untargeted metabolomics LC-MS/MS full-scan mzML data (e.g. serum or beer extract injections) into per-peak bounding boxes of min/max RT and m/z to define which ions any DDA strategy could fragment.

## When NOT to use

- Input is already a peak table or feature list — this skill operates on raw full-scan mzML, not on pre-computed features.
- The goal is peak picking on MS2 fragmentation spectra rather than the MS1 full-scan survey data.
- The instrument acquisition is DIA rather than DDA, where the TopN-driven MS2-scan-slot bounding-box semantics do not apply.

## Inputs

- full-scan MS1 mzML files for the sample extracts (serum and beer)
- MZmine 2 peak-picking parameters (Supporting Information Table SI-1)
- TopN mzML file (for extracting mean MS1/MS2 scan times and N when coupling to coverage analysis)

## Outputs

- per-peak bounding boxes (min/max retention time and m/z) exported from MZmine 2
- picked-peak set defining the fragmentable-ion universe for ViMMS simulations and maximum-matching coverage analysis
- optimal_coverage_comparison.csv (when combined with the matching workflow)

## How to apply

Pick peaks from the full-scan mzML files of the sample extracts using MZmine 2 with the parameters specified in Supporting Information Table SI-1, and export each picked peak as a bounding box of minimum and maximum retention time and m/z. These bounding boxes are the inputs to the bipartite graph construction: a peak can be matched to an MS2 scan only when the MS2 scan time falls within its RT limits, the preceding MS1 survey scan is also within the box, and the peak intensity in that MS1 scan exceeds the minimum MS1 intensity for fragmentation. The same picked-peak boxes feed ViMMS simulations of new DDA methods, so peak-picking parameters directly constrain both theoretical coverage upper bounds (1542 serum, 2955 beer) and in silico method comparisons. Validation: exported boxes must reproduce the reported serum and beer peak counts that yield the observed-vs-optimal TopN coverage gap (656 vs 1542 serum; 1046 vs 2955 beer).

## Related tools

- **MZmine 2** (performs the actual peak picking on full-scan mzML files and exports bounding boxes of min/max RT and m/z per picked peak)
- **ViMMS** (consumes the picked-peak bounding boxes to simulate the virtual metabolomics mass spectrometer and evaluate DDA strategies in silico)

## Examples

```
MZmine2 --batch peak_picking_batch.xml --input serum_fullscan.mzML --output serum_peaks_boxes.csv
```

## Evaluation signals

- Exported bounding boxes must reproduce the reported optimal coverage counts of 1542 peaks for serum and 2955 for beer when fed into the Hopcroft–Karp maximum-matching step.
- Observed TopN coverage derived from the same boxes must be 656 (serum) and 1046 (beer), confirming the picking parameters match SI Table SI-1.
- Every exported box must satisfy the edge-condition semantics: MS2 scan time within RT limits, preceding MS1 scan within the box, and peak intensity exceeding the minimum MS1 intensity for fragmentation.
- Boxes must be consistent with the MZmine 2 parameters in Supporting Information Table SI-1 — re-running with identical parameters should yield identical box counts.
- The resulting boxes must be loadable into ViMMS without conversion errors for downstream DDA strategy simulation.

## Limitations

- Coverage upper bounds depend on the specific MZmine 2 parameters in SI Table SI-1; different peak-picking settings would shift both observed and optimal counts.
- The bounding-box RT/mz limits assume the picked peak is fragmentable only within its elution window and that intensity exceeds the minimum MS1 threshold — peaks with weak or intermittent MS1 signal may be missed.
- Results are grounded in serum and beer extract samples; other matrix types with different ion densities may not transfer the reported counts directly.

## Evidence

- [supplementary] Pick peaks from the full-scan mzML files for the serum and beer extracts using MZmine 2 (parameters per Supporting Information Table SI-1) and export each peak as a bounding box of min/max retention time and m/z.: [evidence span withheld — anchor status: not_found]
- [abstract] an in silico framework for fast and cost-efficient development of new DDA strategies: "an in silico framework for fast and cost-efficient development of new DDA strategies"
- [intro] optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),10: "optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),10"
