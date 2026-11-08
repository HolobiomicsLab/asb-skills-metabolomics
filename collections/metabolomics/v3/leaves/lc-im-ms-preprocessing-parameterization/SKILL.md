---
name: lc-im-ms-preprocessing-parameterization
description: Use when raw profile-mode LC-IM-MS TOF data (Agilent MassHunter .d or
  UIMF) shows background noise, isolated spike points, or detector-saturated peaks
  and must be filtered/smoothed before feature finding or lipid annotation.
when_to_use_negative:
- Input is already centroided or already a feature/annotation table — the algorithms
  operate at the raw data point stage, not downstream results
- Data are not LC-IM-MS (no IM dimension) — spike removal and smoothing are parameterized
  against adjacent points in both IM and MS dimensions
- The goal is to preserve every low-count data point — the intensity threshold (e.g.
  20 counts) deliberately deletes background points to shrink file size
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: PNNL PreProcessor
  role: Standalone software that implements the multidimensional smoothing, low intensity
    threshold, spike removal, and saturation repair algorithms and writes filtered
    MS-files in the same instrument format as the input
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: 'Downstream consumer used to quantify the benefit of the parameterized preprocessing:
    it directly processes the PreProcessor-generated Agilent .d MS-files for feature
    finding and lipid annotation'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:95063cc21a92ba08be1e8a97273c300cd5a83b08474b623e6c9db26d5ed6fb1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lc-im-ms-preprocessing-parameterization

## Summary

Parameterization and application of PNNL PreProcessor noise-filtering algorithms (low intensity threshold, spike removal, multidimensional smoothing, saturation repair) on raw profile-mode LC-IM-MS data to reduce file size and extend dynamic range before downstream omics feature finding.

## When to use

Use when raw profile-mode LC-IM-MS TOF data (Agilent MassHunter .d or UIMF) shows background noise, isolated spike points, or detector-saturated peaks and must be filtered/smoothed before feature finding or lipid annotation.

## When NOT to use

- Input is already centroided or already a feature/annotation table — the algorithms operate at the raw data point stage, not downstream results
- Data are not LC-IM-MS (no IM dimension) — spike removal and smoothing are parameterized against adjacent points in both IM and MS dimensions
- The goal is to preserve every low-count data point — the intensity threshold (e.g. 20 counts) deliberately deletes background points to shrink file size

## Inputs

- Raw or smoothed profile-mode LC-IM-MS MS-file (Agilent MassHunter .d or UIMF), e.g. from MassIVE MSV000087295
- User-specified low intensity threshold in counts (e.g. 20 counts for the lipid workflow)
- Spike-removal adjacent-neighbor criterion (e.g. 1 adjacent point in IM and MS dimensions)

## Outputs

- Filtered MS-file in profile mode (no centroiding) in the same instrument format as the input (.d or UIMF)
- Log file recording the applied algorithms and parameters

## How to apply

Load the raw or smoothed LC-IM-MS MS-file (Agilent MassHunter .d or UIMF profile-mode data, e.g. MassIVE MSV000087295) into the PNNL PreProcessor. First apply the low intensity threshold, removing all raw data points that do not exceed a user-specified cutoff — 20 counts in the lipid workflow, 50 counts in the demultiplexing workflow — so threshold choice must match the analysis goal. Then apply spike removal by comparing each remaining point against adjacent points in the IM and MS dimensions and deleting isolated 'spike' points that lack the required neighbors, using 1 adjacent point as the threshold criterion. Optionally combine with demultiplexing (75% pulses for artifact removal), multidimensional smoothing (points: 3 in LC and 7 in IM, with LC compression 4 frames→1 and IM interpolation each drift bin→3), and saturation repair with a threshold of 40% of the abundance limit for truncated high-abundance peaks. Write the filtered spectra in profile mode (without centroiding) to a new MS-file in the same instrument format as the input, without overwriting the original data, and record the applied algorithms and parameters in a log file.

## Related tools

- **PNNL PreProcessor** (Standalone software that implements the multidimensional smoothing, low intensity threshold, spike removal, and saturation repair algorithms and writes filtered MS-files in the same instrument format as the input) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream consumer used to quantify the benefit of the parameterized preprocessing: it directly processes the PreProcessor-generated Agilent .d MS-files for feature finding and lipid annotation)

## Evaluation signals

- Filtered_ms_file (e.g. filtered_ms_file.uimf) exists and parses as a valid instrument-format MS-file in profile mode, with the processing script running to completion
- Filtered file is smaller than its input, since background points below the intensity threshold and isolated spikes are removed
- Downstream benefit in line with reported metrics: 70.7% more lipid features consistently detected across 3 replicates after smoothing, and 19.4% more lipid annotations for features found in at least 2 replicates
- Saturation-repaired data show reduced mass error (e.g. from 10.6 to 1.9 pm) and increased monoisotopic peak height
- Log file records the exact algorithms and parameters applied (threshold counts, adjacent-point criterion, smoothing window points in LC and IM)

## Limitations

- Filter parameters are workflow-specific and user-specified (20 counts in the lipid workflow vs 50 counts plus 75%-pulse demultiplexing in the demultiplexing workflow); a mismatched threshold can delete real low-abundance signal
- For non-native instrument formats, data must be converted to UIMF as an intermediate step before the algorithms can be applied
- The paper does not specify a PreProcessor tool version, and the filters reduce file size by discarding data points, which is irreversible in the filtered output

## Evidence
