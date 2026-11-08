---
name: lc-ms-peak-picking-bounding-box-extraction
description: Use when you have full-scan mzML peak lists plus mzML files from DDA
  (TopN) controller injections and need per-sample coverage (peaks fragmented) and
  efficiency metrics for LC-MS/MS metabolomics.
when_to_use_negative:
- Input is already a feature table or quantification matrix with no MS2 scan-to-peak
  mapping possible — this skill requires raw full-scan plus MS2 mzML files.
- Goal is evaluating DIA acquisition, where MS2 spectra are not tied to individually
  picked MS1 peaks and event-to-bounding-box matching does not apply.
- Peaks were picked with different parameters than SI Table SI-1 and you intend to
  compare against the paper's reported Table 1/Table 2 values.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Peak-picks chromatographic peaks from full-scan mzML files using SI Table
    SI-1 parameters; exports picked peaks as bounding boxes for matching against MS2
    events.
- name: ViMMS
  role: Virtual metabolomics mass spectrometer framework used to simulate controller
    injections (e.g. TopN) and generate the mzML files whose MS2 events are matched
    to bounding boxes.
- name: XCMS
  role: Alternative peak-detection tool for full-scan mzML files in this workflow.
- name: Peakonly
  role: Alternative peak-detection tool for full-scan mzML files in this workflow.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:69e9ebfdae7c6995038513fd6092f85186e2d4e4b06f0ceda7c6b050ba98ef55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - Peakonly
  - ViMMS
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lc-ms-peak-picking-bounding-box-extraction

## Summary

Extracts picked-peak bounding boxes from full-scan mzML files and intersects them with MS2 fragmentation events from controller injections to compute DDA coverage and efficiency per sample. This is the ground-truth measurement the paper uses to quantify how far standard TopN DDA falls short of the theoretically optimal coverage for a given injection.

## When to use

Use when you have full-scan mzML peak lists plus mzML files from DDA (TopN) controller injections and need per-sample coverage (peaks fragmented) and efficiency metrics for LC-MS/MS metabolomics.

## When NOT to use

- Input is already a feature table or quantification matrix with no MS2 scan-to-peak mapping possible — this skill requires raw full-scan plus MS2 mzML files.
- Goal is evaluating DIA acquisition, where MS2 spectra are not tied to individually picked MS1 peaks and event-to-bounding-box matching does not apply.
- Peaks were picked with different parameters than SI Table SI-1 and you intend to compare against the paper's reported Table 1/Table 2 values.

## Inputs

- Full-scan mzML files of serum (QCA) and beer (QCB) extracts for peak picking
- mzML files from TopN DDA controller injections containing MS2 fragmentation events
- Peak-picking parameters (SI Table SI-1) for MZmine2

## Outputs

- Picked-peak bounding boxes defined by min/max RT and m/z, with RT range from the first and last MS1 scans of each chromatographic peak
- Per-sample TopN coverage and efficiency values plus MS1/MS2 scan counts written to results_table.tsv

## How to apply

Load the full-scan mzML files (e.g. serum QCA and beer QCB extracts) and pick peaks with MZmine2 using the parameters in SI Table SI-1; export each picked peak as a bounding box whose RT range is bounded by the first and last MS1 scans comprising the chromatographic peak and whose m/z range is given by min/max m/z. Read the mzML files from the TopN controller injections and extract the MS2 fragmentation events, then match each MS2 event to the bounding box(es) it falls into (a peak counts as fragmented only if at least one MS2 event lands inside it). Compute coverage as the number of picked peaks containing at least one fragmentation event and efficiency as the ratio of fragmented peaks to the number of MS2 scans. The rationale is that coverage relative to the theoretically optimal performance for that injection quantifies how much room improved DDA prioritization strategies have over standard TopN, which typically fragments only a small subset of ions. Write per-sample TopN coverage and efficiency, alongside MS1 and MS2 scan counts, to results_table.tsv.

## Related tools

- **MZmine 2** (Peak-picks chromatographic peaks from full-scan mzML files using SI Table SI-1 parameters; exports picked peaks as bounding boxes for matching against MS2 events.)
- **ViMMS** (Virtual metabolomics mass spectrometer framework used to simulate controller injections (e.g. TopN) and generate the mzML files whose MS2 events are matched to bounding boxes.)
- **XCMS** (Alternative peak-detection tool for full-scan mzML files in this workflow.)
- **Peakonly** (Alternative peak-detection tool for full-scan mzML files in this workflow.)

## Evaluation signals

- Reported TopN coverage matches Table 1: 656 fragmented picked peaks observed for serum (vs 1542 theoretical optimum) and 1046 for beer (vs 2955).
- Per-sample MS1 and MS2 scan counts match the values in Table 2.
- Every bounding box's RT range spans exactly the first and last MS1 scans comprising its chromatographic peak.
- Efficiency (fragmented peaks / MS2 scans) is at most 1 and consistent with the coverage and scan-count values in results_table.tsv.

## Limitations

- Standard DDA typically fragments only a small subset of ions present, so observed coverage will be well below the theoretical optimum; the paper's improvement estimates depend on the uneven elution distribution of ions being taken into account when computing the optimum.
- Coverage depends on the peak-picking parameters (SI Table SI-1); results are not comparable if peaks are picked with different tools or settings.

## Evidence

- [methods] Load the full-scan mzML files for the serum (QCA) and beer (QCB) extracts and pick peaks with MZmine2 using the parameters specified in SI Table SI-1.: [evidence span withheld — anchor status: not_found]
- [methods] Bounding-box export definition: [evidence span withheld — anchor status: not_found]
- [methods] Coverage computation: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation for coverage measurement: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] Theoretical optimum baseline: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
