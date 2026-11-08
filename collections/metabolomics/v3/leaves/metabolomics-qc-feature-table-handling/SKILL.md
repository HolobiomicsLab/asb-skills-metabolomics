---
name: metabolomics-qc-feature-table-handling
description: Use when raw peak-picked feature tables from two LC–MS metabolomics datasets
  must be QC-filtered and summarized (RT, m/z, FI medians) before feature correspondence
  matching.
when_to_use_negative:
- Input is already a post-QC feature table of RT/mz/FI medians ready for M2S matching
- The goal is metabolite annotation or database searching (LIPID MAPS, METLIN, HMDB,
  GNPS, MassBank) rather than preparing features for correspondence matching
- Datasets were not peak-picked separately with XCMS and lack per-sample feature intensities
  needed for QC and median computation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: Peak-picking of each dataset separately to produce feature tables prior to
    QC
- name: MATLAB
  role: Environment in which the QC-filtered feature tables are loaded and characterized
    for M2S matching
  repo: https://github.com/rjdossan/M2S
- name: M2S (Match 2 Sets)
  role: Downstream toolbox consuming the QC-filtered feature tables for correspondence
    matching
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:ff00e4148a8c08ec4580155e88016deba5236cfc5b251f448151ca706e41a5c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolomics-qc-feature-table-handling

## Summary

Prepares post-XCMS untargeted LC–MS feature tables for inter-dataset matching by applying QC dilution-series filtering (R² < 0.7), retention-time trimming, and per-feature median characterization of RT, m/z, and FI so that only well-behaved features enter correspondence matching.

## When to use

Use when raw peak-picked feature tables from two LC–MS metabolomics datasets must be QC-filtered and summarized (RT, m/z, FI medians) before feature correspondence matching.

## When NOT to use

- Input is already a post-QC feature table of RT/mz/FI medians ready for M2S matching
- The goal is metabolite annotation or database searching (LIPID MAPS, METLIN, HMDB, GNPS, MassBank) rather than preparing features for correspondence matching
- Datasets were not peak-picked separately with XCMS and lack per-sample feature intensities needed for QC and median computation

## Inputs

- Raw LC–MS peak-picked feature tables from XCMS (per-sample RT, m/z, feature intensity values)
- QC dilution series sample annotations for assessing feature dilution response (R²)

## Outputs

- Post-QC feature table with per-feature medians of RT, m/z, and FI across all samples (e.g. 10427 features for MESA, 14097 for Rotterdam in Dataset 1)
- Optionally, a further-filtered table after Step 3 removal of poor matches

## How to apply

Peak-pick each dataset separately with XCMS, then apply QC using a QC dilution series to eliminate features whose dilution response is worse than R² < 0.7. Trim retention time (e.g. keep only features eluting before 12 min, or restrict to 0.45–9.5 min where applicable) to remove poorly behaved elution regions. For each surviving feature, compute the medians of RT, m/z, and FI across all samples, since these medians are the values consumed by the M2S Step 1 candidate matching and Step 2 unique-correspondence scoring. Optionally apply Step 3 tightening (e.g. a tighter upper FI threshold to remove column-bleeding clusters) after inspecting distance plots, but note that Step 3 is an analyst decision and may delete correct matches. On the MESA/Rotterdam example this QC pipeline yielded 10427 (MESA) and 14097 (Rotterdam) features for Dataset 1, and 6793/6315 for Dataset 2.

## Related tools

- **XCMS** (Peak-picking of each dataset separately to produce feature tables prior to QC)
- **MATLAB** (Environment in which the QC-filtered feature tables are loaded and characterized for M2S matching) — https://github.com/rjdossan/M2S
- **M2S (Match 2 Sets)** (Downstream toolbox consuming the QC-filtered feature tables for correspondence matching) — https://github.com/rjdossan/M2S

## Evaluation signals

- Feature counts after QC should be recorded and comparable to reported values (e.g. Dataset 1: 10427 MESA / 14097 Rotterdam features after QC)
- Every retained feature must have non-missing medians of RT, m/z, and FI across all samples
- All retained features should pass the QC dilution-series threshold (dilution response R² ≥ 0.7)
- No retained features should elute outside the trimmed RT window (e.g. before 12 min retention trimming)
- Downstream M2S Step 1 candidate matching should produce plausible distance plots, with tight thresholds removable in Step 3 only for clearly anomalous clusters (e.g. column bleeding)

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on whether to use step 3 (poor match deletion) rests with the analyst, and this optional step may inadvertently delete some correct matches
- Proper QC is essential: skipping it leads to poor matches, and the M2S toolbox ships without a stated software version

## Evidence

- [methods] a QC dilution series was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7.: "a QC dilution
series14 was used to eliminate features
whose response to dilution was worse than a threshold of R2 <0.7."
- [methods] only features eluting before 12 min were retained: "only features eluting before 12 min were retained"
- [methods] the medians of RT, m/z, and FI across all samples were used.: "the medians of RT, m/z, and FI across all samples were used."
- [methods] Both datasets were (separately) peak-picked using XCMS: "Both datasets were (separately) peak-picked
using XCMS,13"
- [discussion] Proper quality control of features prior to matching avoids poor matches: "Proper
quality control of features prior to matching avoids poor matches"
- [readme] Matlab package to match untargeted metabolomic features of two LC-MS datasets: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
