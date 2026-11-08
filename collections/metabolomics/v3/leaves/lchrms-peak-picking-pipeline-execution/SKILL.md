---
name: lchrms-peak-picking-pipeline-execution
description: Use when raw LC-HRMS profile-mode chromatograms must be converted into
  chromatographic feature lists, or when comparing peak-picking tools (PeakBot vs
  XCMS, MS-Dial, peakOnly) on a shared dataset such as the PHM porcine hepatic microsome
  data.
when_to_use_negative:
- Input is already a centroided feature table or peak list — only alignment/grouping
  is needed, not profile-mode peak picking.
- Fewer than 100 reference features are available and no pretrained PeakBot model
  exists — the CNN cannot be trained to high quality.
- Data are not profile-mode LC-HRMS chromatograms (e.g. GC-MS or pre-processed centroided
  data), which the local-maxima-to-standardized-area workflow does not target.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: 'Primary peak-picking engine: local-maxima extraction into standardized rt
    x mz areas, CNN classification, and feature grouping/alignment'
  repo: https://github.com/christophuv/PeakBot
- name: XCMS
  role: Comparison peak-picking pipeline run on the same dataset to produce an independent
    feature list
- name: MS-Dial
  role: Comparison peak-detection tool producing a feature list for the four-tool
    Venn overlap
- name: peakOnly
  role: Comparison peak-detection tool producing a feature list for the four-tool
    Venn overlap
- name: TensorFlow
  role: Implements the CNN model (version 2.5.0) that classifies standardized areas
    as chromatographic peaks or background
  repo: https://github.com/christophuv/PeakBot
- name: Anaconda
  role: Recommended virtual-environment platform for installation so CUDA dependencies
    are installed automatically
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:e57d12104d8f383436e598a8748002c59eb145e7b9a76b45e958ea9313134c94
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - Anaconda
  - MS-Dial
  - PeakBot
  - Python
  - TensorFlow
  - XCMS
  - peakOnly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# lchrms-peak-picking-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes an end-to-end LC-HRMS profile-mode peak-picking pipeline (PeakBot CNN-based detection, or equivalent tools such as XCMS, MS-Dial and peakOnly) to convert raw chromatograms into aligned feature lists, and quantifies cross-tool feature overlap as validation.

## When to use

Use when raw LC-HRMS profile-mode chromatograms must be converted into chromatographic feature lists, or when comparing peak-picking tools (PeakBot vs XCMS, MS-Dial, peakOnly) on a shared dataset such as the PHM porcine hepatic microsome data.

## When NOT to use

- Input is already a centroided feature table or peak list — only alignment/grouping is needed, not profile-mode peak picking.
- Fewer than 100 reference features are available and no pretrained PeakBot model exists — the CNN cannot be trained to high quality.
- Data are not profile-mode LC-HRMS chromatograms (e.g. GC-MS or pre-processed centroided data), which the local-maxima-to-standardized-area workflow does not target.

## Inputs

- Profile-mode LC-HRMS chromatograms (e.g. the PHM porcine hepatic microsome dataset)
- Trained PeakBot CNN model (or a newly trained one built from >=100 reference features)
- Reference feature TSV with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation
- Wall background TSV (MZ, Rtstart, Rtend, MZdeviation) and other-background TSV (RTStart, RTEnd, MZLow, MZHigh) for training
- Feature lists from comparison tools (XCMS, MS-Dial, peakOnly) for overlap analysis

## Outputs

- Detected chromatographic feature list per sample (peak apex, bounding box, peak center)
- Grouped feature data matrix and featureML file
- Aligned cross-tool overlap statistics written to peakbot_overlap_stats.tsv (four-tool Venn percentages)

## How to apply

Install PeakBot in an Anaconda virtual environment (Python 3.8, TensorFlow 2.5.0; with a CUDA-enabled Nvidia GPU plus CUDA toolkit and cuDNN for training/GPU acceleration), and estimate the chromatogram-specific settings (noiseLevel, minIntensity, minRT/maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM) via estimateParameters.py. Run peak detection: local-maxima detection extracts super-sampled standardized areas (retention-time versus m/z) which a custom-trained CNN classifies as peak apex versus background, also predicting a peak center and bounding box; feature grouping across samples uses peakbot.cuda.KNNalignFeatures and peakbot.cuda.groupFeatures to produce a data matrix and featureML file. For benchmarking, run XCMS, MS-Dial and peakOnly independently on the same raw files, then align all four feature lists by retention time and m/z and compute Venn-overlap statistics. Validate against the reported PHM benchmark: ~73% of features detected by all four tools, ~12% missed only by peakOnly, ~4.7% exclusive to MS-Dial/peakOnly/PeakBot, and a 94.65% total detection rate for PeakBot, writing results to peakbot_overlap_stats.tsv.

## Related tools

- **PeakBot** (Primary peak-picking engine: local-maxima extraction into standardized rt x mz areas, CNN classification, and feature grouping/alignment) — https://github.com/christophuv/PeakBot
- **XCMS** (Comparison peak-picking pipeline run on the same dataset to produce an independent feature list)
- **MS-Dial** (Comparison peak-detection tool producing a feature list for the four-tool Venn overlap)
- **peakOnly** (Comparison peak-detection tool producing a feature list for the four-tool Venn overlap)
- **TensorFlow** (Implements the CNN model (version 2.5.0) that classifies standardized areas as chromatographic peaks or background) — https://github.com/christophuv/PeakBot
- **Anaconda** (Recommended virtual-environment platform for installation so CUDA dependencies are installed automatically) — https://github.com/christophuv/PeakBot

## Examples

```
pip install git+https://github.com/christophuv/PeakBot
```

## Evaluation signals

- Four-tool Venn overlap on the PHM dataset reaches ~73% of features detected by PeakBot, XCMS, MS-Dial and peakOnly simultaneously.
- Per-tool discrepancy percentages match reported values: ~12% features missed only by peakOnly and ~4.7% exclusive to MS-Dial/peakOnly/PeakBot.
- PeakBot total detection rate across all features is ~94.65%.
- CNN discrimination of chromatographic peaks vs background reaches accuracy ~0.99 on training and independent validation datasets.
- Output featureML/data-matrix schema is produced and reference feature TSVs conform to the required columns (RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation).

## Limitations

- Training a new model requires a CUDA-enabled Nvidia graphics card plus CUDA toolkit and cuDNN libraries; without GPU the instance-generation approach is slow.
- A minimum of 100 reference features (isotopologs allowed) is needed, and care must be taken that no co-eluting isomers are present in reference chromatograms; different peak forms (fronting, tailing) must be represented.
- intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to evaluate and may require visual tools like TOPPView or the estimateParameters.py helper.

## Evidence

- [readme] peakbot.cuda.preProcessChromatogram` for extracting the standardized areas from the chromatogram and `peakbot.runPeakBot`: [evidence span withheld — anchor status: external_doc]
