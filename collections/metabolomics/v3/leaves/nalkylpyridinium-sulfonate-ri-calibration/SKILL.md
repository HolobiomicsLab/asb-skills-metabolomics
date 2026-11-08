---
name: nalkylpyridinium-sulfonate-ri-calibration
description: Use when retention times of features measured in one lab must be matched
  to reference compounds annotated in another (e.g. HPLC vs UHPLC), so annotation
  should rely on m/z plus retention index rather than raw RT.
when_to_use_negative:
- The reference compounds and features were measured under the same chromatographic
  conditions in the same laboratory, so raw retention times can be matched directly
  (e.g. MzRtParam on RT).
- No retention-index standard series was acquired alongside the samples, so RT-to-RI
  conversion cannot be calibrated.
- The goal is MS2-based spectral annotation rather than MS1 m/z + retention-based
  annotation.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaboCoreUtils
  role: Provides the indexRtime() function that converts feature retention times to
    retention indices by interpolation against the NAP standards
  repo: https://rformassspectrometry.github.io/MetaboCoreUtils/
- name: MetaboAnnotation
  role: Consumes the computed retention indices via Mass2MzRtParam (toleranceRt in
    RI units) for m/z + RI annotation with matchMz()
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: MetaboAnnotationTutorials
  role: Distributes the NAP standards, feature table, and HPLC/UHPLC RI reference
    databases used for calibration and annotation
  repo: https://github.com/jorainer/MetaboAnnotationTutorials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:4a9038b3d7ec28a34074a8afee1e669636809b2ed238f127b220f250be6819fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnnotation
  - MetaboAnnotationTutorials
  - MetaboCoreUtils
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# nalkylpyridinium-sulfonate-ri-calibration

## Summary

Converts raw LC-MS retention times into laboratory-independent retention indices by linear interpolation against a series of N-Alkyl-pyridinium sulfonate (NAP) retention-index standards, using MetaboCoreUtils::indexRtime(). Retention indices then serve as the retention coordinate for cross-laboratory annotation with m/z + RI matching instead of raw retention times.

## When to use

Use when retention times of features measured in one lab must be matched to reference compounds annotated in another (e.g. HPLC vs UHPLC), so annotation should rely on m/z plus retention index rather than raw RT.

## When NOT to use

- The reference compounds and features were measured under the same chromatographic conditions in the same laboratory, so raw retention times can be matched directly (e.g. MzRtParam on RT).
- No retention-index standard series was acquired alongside the samples, so RT-to-RI conversion cannot be calibrated.
- The goal is MS2-based spectral annotation rather than MS1 m/z + retention-based annotation.

## Inputs

- MS1 feature table with retention times (rti_ms_pos_features.txt from MetaboAnnotationTutorials)
- N-Alkyl-pyridinium sulfonate retention-index standards table (rti_ms_pos_naps.txt)
- Optional reference databases with retention indices for matching (rti_db_hplc.txt, rti_db_uhplc.txt)

## Outputs

- Feature table with an added retention-index column (rindex)
- RT vs retention-index calibration relationship for the NAP standards
- RI-based annotation results when combined with Mass2MzRtParam/matchMz matching

## How to apply

Load the MS1 feature table (rti_ms_pos_features.txt) and the NAP standards (rti_ms_pos_naps.txt) from the MetaboAnnotationTutorials package and plot RT vs retention index to confirm the calibration relationship; the index is defined as the number of alkyl-chain carbons x 100. Call MetaboCoreUtils::indexRtime() with default linear interpolation, passing feature retention times and the NAP standard table, to append a retention-index column to the features. Use the resulting indices in MetaboAnnotation::Mass2MzRtParam with adducts [M+H]+ and [M+Na]+, m/z tolerance 0.005 Da, toleranceRt of 10 retention-index units, and column mappings massColname='exact.mass', mzColname='mz', rtColname='rindex'. Because retention times are only partially transferable between laboratories, the RI conversion is what makes matching against external HPLC (rti_db_hplc.txt) and UHPLC (rti_db_uhplc.txt) reference databases feasible.

## Related tools

- **MetaboCoreUtils** (Provides the indexRtime() function that converts feature retention times to retention indices by interpolation against the NAP standards) — https://rformassspectrometry.github.io/MetaboCoreUtils/
- **MetaboAnnotation** (Consumes the computed retention indices via Mass2MzRtParam (toleranceRt in RI units) for m/z + RI annotation with matchMz()) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **MetaboAnnotationTutorials** (Distributes the NAP standards, feature table, and HPLC/UHPLC RI reference databases used for calibration and annotation) — https://github.com/jorainer/MetaboAnnotationTutorials

## Examples

```
ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps)
```

## Evaluation signals

- The retention time vs retention index relationship of the NAP standards is monotonic after indexRtime conversion.
- Every feature in the input table receives a non-missing rindex value within the calibrated range of the NAP standard series.
- Downstream Mass2MzRtParam matching (m/z tolerance 0.005, toleranceRt 10 RI units) returns matches for a plausible subset of features, verified via whichQuery().
- matchedData() contains the expected columns (Name, mz, rindex, target_name, adduct, score, score_rt) and the output CSV is a valid, non-empty table.

## Limitations

- Retention times are only partially transferable between laboratories, which is the motivation for RI conversion; raw RT matching across labs remains unreliable.
- Interpolation is only valid within the retention-time range spanned by the NAP standards; features outside this range extrapolate poorly.
- The linear index definition (alkyl-chain carbons x 100) depends on the NAP standard series being measured in the same run conditions as the features.

## Evidence

- [methods] We can use the N-Alkly-pyridinium sulfonates to convert the retention times of the measured features using the `indexRtime` function.: "We can use the N-Alkly-pyridinium sulfonates to convert the retention times of the measured features using the `indexRtime` function."
- [methods] ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps): "ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps)"
- [methods] Considering also the retention times would allow a better annotation and increase its confidence.: "Considering also the retention times would allow a better annotation and increase its confidence."
- [readme] This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data: "This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data"
