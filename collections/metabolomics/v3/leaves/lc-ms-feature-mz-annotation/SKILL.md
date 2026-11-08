---
name: lc-ms-feature-mz-annotation
description: Use when you have a pre-processed MS1 feature table (m/z + retention
  time) and a compound reference database with exact masses and RTs, and retention
  times are only partially transferable between labs or methods.
when_to_use_negative:
- You have raw MS2 spectra and want library matching — use matchSpectra-based MS2
  annotation instead of MS1 m/z(+RT) matching
- Your reference database lacks exact masses and retention information (e.g. name-only
  lists) — Mass2MzRtParam cannot be parameterized
- Input is raw instrument data rather than a picked, isotope-grouped feature table
  — peak detection/refinement must happen first
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnnotation
  role: Provides matchMz() with Mass2MzRtParam/Mass2MzParam for m/z(+RI) annotation
    against reference databases, and whichQuery() to subset matched features
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: MetaboCoreUtils
  role: Provides indexRtime() to convert feature retention times to retention indices
    against RTI standards
  repo: https://rformassspectrometry.github.io/MetaboCoreUtils/
- name: MetaboAnnotationTutorials
  role: Supplies the example feature table, NAP standards and HPLC/UHPLC reference
    databases used in the use case
  repo: https://github.com/jorainer/MetaboAnnotationTutorials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:508cbc81f8b7559e15ef789b89089fe9baa1ff49a9627e6af548b6bbf16cf098
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnnotation
  - MetaboAnnotationTutorials
  - MetaboCoreUtils
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lc-ms-feature-mz-annotation

## Summary

Annotates LC-MS1 features by matching their m/z (with adducts [M+H]+ and [M+Na]+) against reference compound databases, optionally using retention times converted to retention indices via indexRtime so annotations transfer between HPLC/UHPLC methods.

## When to use

Use when you have a pre-processed MS1 feature table (m/z + retention time) and a compound reference database with exact masses and RTs, and retention times are only partially transferable between labs or methods.

## When NOT to use

- You have raw MS2 spectra and want library matching — use matchSpectra-based MS2 annotation instead of MS1 m/z(+RT) matching
- Your reference database lacks exact masses and retention information (e.g. name-only lists) — Mass2MzRtParam cannot be parameterized
- Input is raw instrument data rather than a picked, isotope-grouped feature table — peak detection/refinement must happen first

## Inputs

- Pre-processed MS1 feature table with m/z and retention times (e.g. rti_ms_pos_features.txt from MetaboAnnotationTutorials)
- Retention-index standards (e.g. N-Alkyl-pyridinium sulfonates, rti_ms_pos_naps.txt)
- Reference compound database(s) with exact mass, m/z and retention index/RT columns (e.g. rti_db_hplc.txt, rti_db_uhplc.txt)

## Outputs

- Annotated feature table with matched compound names, m/z, retention index, adduct, m/z score and RT/RI score (e.g. rti_annotation_results.csv)
- Retention-index-annotated feature table (feature rindex column from indexRtime)

## How to apply

Load the MS1 feature table (e.g. rti_ms_pos_features.txt) and retention-index standards such as N-Alkyl-pyridinium sulfonates (rti_ms_pos_naps.txt) from MetaboAnnotationTutorials, and verify the RT-vs-RI relationship of the standards is monotonic. Convert feature retention times to retention indices with MetaboCoreUtils::indexRtime() (linear interpolation; RI = alkyl-chain carbons x 100 for NAPs), because raw RTs are only partially transferable between laboratories while indices are not. Configure MetaboAnnotation::Mass2MzRtParam with adducts c('[M+H]+', '[M+Na]+'), m/z tolerance 0.005 Da, and toleranceRt of 10 retention-index units, mapping massColname='exact.mass', mzColname='mz', rtColname='rindex'. Run matchMz() of the features against each reference database (e.g. rti_db_hplc.txt and rti_db_uhplc.txt) separately, then subset with whichQuery() to keep only features with at least one match. Extract matchedData() columns (Name, mz, rindex, target_name, adduct, score, score_rt) and combine into a single annotated table. The rationale is that m/z alone matches most peaks to several compounds, so adding retention (index) constraints increases annotation confidence.

## Related tools

- **MetaboAnnotation** (Provides matchMz() with Mass2MzRtParam/Mass2MzParam for m/z(+RI) annotation against reference databases, and whichQuery() to subset matched features) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **MetaboCoreUtils** (Provides indexRtime() to convert feature retention times to retention indices against RTI standards) — https://rformassspectrometry.github.io/MetaboCoreUtils/
- **MetaboAnnotationTutorials** (Supplies the example feature table, NAP standards and HPLC/UHPLC reference databases used in the use case) — https://github.com/jorainer/MetaboAnnotationTutorials

## Examples

```
ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps); match_param <- Mass2MzRtParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0.005, toleranceRt = 10)
```

## Evaluation signals

- The output annotation table exists, parses as a valid CSV, and contains Name, mz, rindex, target_name, adduct, score, score_rt columns plus a database indicator
- The retention-index conversion reproduces a monotonic retention time vs retention index relationship for the NAP standards
- Each database match result, subset with whichQuery(), contains only features with at least one match
- m/z matches use tolerance 0.005 Da with adducts [M+H]+ and [M+Na]+, and RT matches use toleranceRt of 10 retention-index units, as parameterized
- Both HPLC and UHPLC reference databases are matched independently and results are combined

## Limitations

- Retention times are only partially transferable between laboratories, motivating retention-index conversion rather than direct RT matching
- Using only m/z values, most chromatographic peaks match several possible compounds, so m/z-only annotation is ambiguous
- The workflow was demonstrated on C. elegans metabolite extract features with HPLC and UHPLC reference databases; other matrix/method combinations need re-validated tolerances

## Evidence

- [methods] Retention time to retention index conversion with indexRtime: "We can use the N-Alkly-pyridinium sulfonates to convert the retention times of the measured features using the `indexRtime` function."
- [methods] Mass2MzRtParam parameters: "match_param <- Mass2MzRtParam(adducts = c("[M+H]+", "[M+Na]+"),
                              tolerance = 0.005,
                              toleranceRt = 10)"
- [methods] m/z-only annotation ambiguity: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] Adding RT improves confidence: "Considering also the retention times would allow a better annotation and increase its confidence."
- [readme] Repository supplying the use case data: "This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data"
