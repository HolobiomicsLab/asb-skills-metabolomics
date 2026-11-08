---
name: xcms-sample-class-annotation
description: Use when an XCMS XCMSnExp/xcmsSet from labeled-vs-unlabeled LC/MS metabolomics
  experiments must have its samples assigned to condition classes (e.g., CELL_Glc12
  unlabeled vs CELL_Glc13 labeled) before geoRge processing.
when_to_use_negative:
- The feature table is already a flat CSV/MGF-style peak list without an xcmsSet object,
  where sampclass() cannot operate.
- The experiment has no labeled/unlabeled biological pairing (e.g., no stable-isotope
  tracer), so class distinction between ULtag/Ltag conditions is unnecessary.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Provides the xcmsSet object and sampclass() mechanism used to annotate sample
    condition classes prior to geoRge analysis
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: geoRge
  role: Consumes the annotated xcmsSet; its PuInc_seeker and label_compare functions
    rely on ULtag/Ltag class labels to detect stable isotope labeling
  repo: https://github.com/jcapelladesto/geoRge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:ccc209e04e642258a439806d59fbbcd1d4225dfb9da419113818a2f7e96f585a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  - mtbls213
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# xcms-sample-class-annotation

## Summary

Annotate XCMS feature-table sample classes (e.g., unlabeled control vs. stable-isotope-labeled conditions) using XCMS's sampclass() so downstream geoRge functions like PuInc_seeker and label_compare can distinguish which biological group each sample belongs to.

## When to use

Use when an XCMS XCMSnExp/xcmsSet from labeled-vs-unlabeled LC/MS metabolomics experiments must have its samples assigned to condition classes (e.g., CELL_Glc12 unlabeled vs CELL_Glc13 labeled) before geoRge processing.

## When NOT to use

- The feature table is already a flat CSV/MGF-style peak list without an xcmsSet object, where sampclass() cannot operate.
- The experiment has no labeled/unlabeled biological pairing (e.g., no stable-isotope tracer), so class distinction between ULtag/Ltag conditions is unnecessary.

## Inputs

- XCMS xcmsSet/XCMSnExp object from peak picking, alignment, and grouping of LC/MS mzXML files
- Metadata mapping sample names to biological conditions (e.g., unlabeled control 'CELL_Glc12', labeled 'CELL_Glc13')

## Outputs

- xcmsSet with annotated sample classes suitable for geoRge PuInc_seeker and label_compare inputs

## How to apply

After running the XCMS pipeline (peak picking, alignment, grouping), assign each sample to its biological condition class using XCMS's sampclass() function on the xcmsSet object, naming classes so the unlabeled control and labeled conditions are clearly separated (e.g., 'CELL_Glc12' for unlabeled and 'CELL_Glc13' for 13C-labeled glucose conditions). These class labels are required inputs for geoRge functions like PuInc_seeker and label_compare, which use ULtag/Ltag strings to match condition names and compare isotopic distributions between unlabeled and labeled biologically equivalent samples. Ensure class tags match the ULtag/Ltag arguments exactly, since geoRge pairs features by comparing mass spectra between the tagged classes.

## Related tools

- **XCMS** (Provides the xcmsSet object and sampclass() mechanism used to annotate sample condition classes prior to geoRge analysis) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **geoRge** (Consumes the annotated xcmsSet; its PuInc_seeker and label_compare functions rely on ULtag/Ltag class labels to detect stable isotope labeling) — https://github.com/jcapelladesto/geoRge

## Examples

```
sampclass(XCMSet) <- c(rep("CELL_Glc12", n_unlabeled), rep("CELL_Glc13", n_labeled))
s1 <- PuInc_seeker(XCMSet = XCMSet, ULtag = "CELL_Glc12", Ltag = "CELL_Glc13", sep.pos.front = TRUE, fc.threshold = 1.5, p.value.threshold = 0.05, PuInc.int.lim = 4000)
```

## Evaluation signals

- xcmsSet contains at least two distinct sample classes corresponding to unlabeled control and labeled conditions.
- sampclass() labels match the ULtag/Ltag strings supplied to PuInc_seeker (e.g., 'CELL_Glc12', 'CELL_Glc13').
- Downstream PuInc_seeker runs without class-name mismatch errors and returns a non-empty result object.
- label_compare produces a table whose fold-change columns are populated with finite values across labeled conditions.

## Limitations

- geoRge relies entirely on XCMS parameters for feature selection, so class annotation quality is bounded by upstream XCMS grouping settings.
- sampclass() labels must be set before running geoRge; a mismatch between class names and ULtag/Ltag arguments will prevent correct pairing of unlabeled and labeled samples.

## Evidence
