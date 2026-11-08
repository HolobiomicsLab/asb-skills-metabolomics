---
name: lcms-ms-map-image-conversion
description: Use when raw LC–MS data (m/z vs retention time) must be converted into
  NPFimg's MS-map image format to enable marker feature identification and produce
  a marker feature table without peak picking.
when_to_use_negative:
- Data is already a processed marker feature table or a tabular feature matrix rather
  than raw two-dimensional LC–MS data requiring map conversion.
- The analysis requires conventional peak picking with XCMS rather than NPFimg's image-based
  approach.
- Input is GC–MS data already formatted as an MS map, where NPFimg can be applied
  directly without an LC–MS conversion step.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NPFimg
  role: Consumes the converted two-dimensional LC–MS map (m/z vs retention time) to
    identify and visualize marker features via image processing and machine learning,
    without peak picking.
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Comparator tool for benchmarking the reliability of NPFimg's signal acquisition
    and marker identification error rates on the converted maps.
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:eb4f095cf63a5291737c0f9fefdb3375a5ad16a298c63aa791ecc20dc1b4bbe6
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lcms-ms-map-image-conversion

## Summary

Convert liquid chromatography–MS data into the two-dimensional NPFimg MS-map image format (m/z vs retention time) so that NPFimg's image-processing and machine-learning pipeline can identify and visualize multivariate chemo-/biomarker features without conventional peak picking.

## When to use

Use when raw LC–MS data (m/z vs retention time) must be converted into NPFimg's MS-map image format to enable marker feature identification and produce a marker feature table without peak picking.

## When NOT to use

- Data is already a processed marker feature table or a tabular feature matrix rather than raw two-dimensional LC–MS data requiring map conversion.
- The analysis requires conventional peak picking with XCMS rather than NPFimg's image-based approach.
- Input is GC–MS data already formatted as an MS map, where NPFimg can be applied directly without an LC–MS conversion step.

## Inputs

- liquid chromatography–MS two-dimensional data (m/z vs retention time)
- marker feature table output file (e.g., marker_features_lcms.csv)

## Outputs

- NPFimg-compatible two-dimensional MS-map image (m/z vs retention time)
- marker feature table (marker_features_lcms.csv) with feature identifier, m/z, retention time, and marker classification/visualization coordinates

## How to apply

Load the liquid chromatography–MS data, which shares the two-dimensional MS-map structure (m/z vs retention time) used by NPFimg for GC–MS, and convert it into the image-based MS-map representation the NPFimg package expects. This avoids the conventional peak picking process, which NPFimg's authors note suffers from false peak detections. Once converted, the image map can be fed directly into NPFimg's image processing and machine learning workflow to discriminate analytes and identify multivariate chemo-/biomarker features. The decision point is whether the LC–MS instrument output can be rendered as a two-dimensional m/z vs retention-time map; if so, no modification of NPFimg's core image processing or machine learning approach is needed. Validation centers on the marker feature table (e.g., marker_features_lcms.csv) being parseable and containing feature identifiers with m/z and retention time coordinates.

## Related tools

- **NPFimg** (Consumes the converted two-dimensional LC–MS map (m/z vs retention time) to identify and visualize marker features via image processing and machine learning, without peak picking.) — github.com/poomcj/NPFimg
- **XCMS** (Comparator tool for benchmarking the reliability of NPFimg's signal acquisition and marker identification error rates on the converted maps.)

## Examples

```
python npfimg_pipeline.py --input lcms_ms_map.csv --output marker_features_lcms.csv
```

## Evaluation signals

- Output marker feature table (e.g., marker_features_lcms.csv) exists and is parseable as a table.
- The table contains the expected feature columns: feature identifier, m/z, retention time, and marker classification/visualization coordinates.
- Marker features are derived without conventional peak picking, i.e., no intermediate peak list from tools like XCMS is used for the conversion step.
- The two-dimensional MS-map image preserves the m/z vs retention-time axes from the original LC–MS data.
- Error rates of signal acquisition and marker identification, when benchmarked against XCMS, should be lower for the NPFimg-derived features.

## Limitations

- NPFimg was demonstrated on GC–MS (aroma odor and human breath) data; its extension to LC–MS is stated as potential applicability rather than a validated case study, so the LC–MS conversion step is an extrapolation of the core approach.
- The README notes documentation is incomplete ('I will update soon'), so exact input file formats and image-conversion parameters are not fully specified in the provided context.
- The approach is limited to two-dimensional MS data (m/z vs retention time); other data structures would not be directly supported.

## Evidence
