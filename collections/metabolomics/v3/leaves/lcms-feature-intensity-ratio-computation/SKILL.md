---
name: lcms-feature-intensity-ratio-computation
description: Use when XCMS peak-picked LC/MS data contains labeled and unlabeled biologically
  equivalent samples (e.g., 12C vs 13C glucose) and you need fold-change intensities
  of enriched features versus a control condition.
when_to_use_negative:
- Input is a fully annotated list of known metabolites where isotopic enrichment has
  already been assigned (e.g., after database_query against ExampleDatabase.csv) —
  label_compare is for untargeted relative quantification of enriched features
- Samples are not biologically equivalent unlabeled/labeled pairs (no stable isotope
  labeling was applied), since comparing isotopic distributions between such samples
  is the premise of the method
- Data has not been processed by XCMS peak picking, alignment, and grouping, so no
  consistent feature table exists to compare intensities across conditions
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: geoRge
  role: Provides the label_compare function that computes labeled-vs-unlabeled intensity
    fold changes; invoked on the output of PuInc_seeker and basepeak_finder
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment, and grouping that produces the XCMSSet feature
    table whose intensities label_compare compares across conditions
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: MSnbase
  role: Dependency for handling XCMSSet sample classes and metadata in R
- name: devtools
  role: Used to install the geoRge package from GitHub before running label_compare
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:d201a078d21d16877ee18cf1c2572b28eca970f572db0b051fe9e31eafea8d90
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - MSnbase
  - R
  - XCMS
  - devtools
  - geoRge
  - mtbls213
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# lcms-feature-intensity-ratio-computation

## Summary

Computes relative-quantification fold changes between unlabeled (control) and stable-isotope-labeled LC/MS features by comparing XCMS-extracted intensities across conditions, as implemented in geoRge's label_compare function.

## When to use

Use when XCMS peak-picked LC/MS data contains labeled and unlabeled biologically equivalent samples (e.g., 12C vs 13C glucose) and you need fold-change intensities of enriched features versus a control condition.

## When NOT to use

- Input is a fully annotated list of known metabolites where isotopic enrichment has already been assigned (e.g., after database_query against ExampleDatabase.csv) — label_compare is for untargeted relative quantification of enriched features
- Samples are not biologically equivalent unlabeled/labeled pairs (no stable isotope labeling was applied), since comparing isotopic distributions between such samples is the premise of the method
- Data has not been processed by XCMS peak picking, alignment, and grouping, so no consistent feature table exists to compare intensities across conditions

## Inputs

- XCMS XCMSSet object with peak-picked, aligned, and grouped LC/MS features (e.g., MetaboLights MTBLS213 mzXML files)
- geoRge result object from PuInc_seeker and basepeak_finder (labelled-feature candidates)
- Sample class assignments separating control (unlabeled) condition from labeled conditions (e.g., CELL_Glc12 vs CELL_Glc13)

## Outputs

- Relative-quantification table of labeled features with m/z, retention time, fold change vs control (fc.vs.Control), and p-value (p.value.vs.Control) columns
- CSV export of labeled-feature fold changes (e.g., label_compare_results.csv)

## How to apply

Load the XCMSSet (e.g., mtbls213 from MetaboLights) together with the upstream geoRge result object produced by basepeak_finder, then assign sample classes with sampclass() distinguishing the unlabeled control condition (ULtag, e.g. CELL_Glc12) from labeled conditions (Ltag, e.g. CELL_Glc13). Invoke label_compare with the geoRge result, the XCMSSet, a mass tolerance of ppm.s=6.5, the control condition name, and fold-change/p-value thresholds (fc.vs.Control, p.value.vs.Control); optionally pass Show.bp to retain base-peak annotations. The function compares intensity distributions of labeled features against unlabeled biologically equivalent samples, because isotopic enrichment manifests as new m/z peaks and higher intensity peaks in labeled samples. Output a relative-quantification table of labeled features with fold changes versus control, which can be written to CSV with m/z, retention time, fold-change, and p-value columns. Validate that fold-change values are finite, plausible, and that features pass the fc.vs.Control threshold as defined in geoRge.

## Related tools

- **geoRge** (Provides the label_compare function that computes labeled-vs-unlabeled intensity fold changes; invoked on the output of PuInc_seeker and basepeak_finder) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment, and grouping that produces the XCMSSet feature table whose intensities label_compare compares across conditions) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **MSnbase** (Dependency for handling XCMSSet sample classes and metadata in R)
- **devtools** (Used to install the geoRge package from GitHub before running label_compare)

## Examples

```
hits <- label_compare(geoRgeR = s2, XCMSet = mtbls213, ppm.s = 6.5, control.cond = "CELL_Glc12", fc.vs.Control = 1.5, p.value.vs.Control = 0.05)
```

## Evaluation signals

- Output CSV exists and contains rows for labeled features with fold-change columns for each labeled condition (e.g., normal vs high glucose vs control)
- Fold-change values are finite and pass the fc.vs.Control threshold specified in the label_compare call (as in geoRge)
- p-value column present and features satisfy p.value.vs.Control significance cutoff
- Sample classes correctly assigned: control (unlabeled) condition distinguished from labeled conditions via sampclass() before comparison
- Reported m/z and RT values align with features carried through from the basepeak_finder geoRge result object

## Limitations

- Requires biologically equivalent unlabeled and labeled samples; comparisons are only meaningful when both conditions were measured under the same LC/MS run
- Depends entirely on XCMS parameters for feature selection, so poor peak picking/alignment/grouping propagates into fold-change results
- The label_compare() function required patching in geoRge 1.0; older versions may misbehave
- Fold-change detection is sensitive to the ppm.s tolerance (6.5 ppm in the example) and the fc.threshold/PuInc.int.lim settings used upstream in PuInc_seeker (e.g., fc.threshold=1.5, p.value.threshold=0.05, PuInc.int.lim=4000)

## Evidence

- [readme] The README documents the geoRge 1.0 patch to label_compare and points users to function help for the new argument set.: [evidence span withheld — anchor status: external_doc]
