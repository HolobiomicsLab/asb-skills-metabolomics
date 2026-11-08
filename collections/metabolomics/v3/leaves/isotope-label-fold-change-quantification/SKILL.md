---
name: isotope-label-fold-change-quantification
description: Use when you have an upstream geoRge result (PuInc_seeker + basepeak_finder)
  plus an XCMS set with control/unlabeled and labeled conditions and need labeled-vs-control
  fold changes and p-values per feature.
when_to_use_negative:
- The data set has no unlabeled biologically equivalent control sample to compare
  against — label_compare depends on comparing isotopic distributions between unlabeled
  and labeled samples
- The upstream geoRge steps (PuInc_seeker, basepeak_finder) have not been run; label_compare
  expects the geoRgeR result object as input
- The goal is theoretical isotope-pattern annotation of a single sample rather than
  empirical labeled-vs-unlabeled comparison — use a different approach (e.g. database_query
  for adduct/compound annotation, not fold-change quantification)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: Provides label_compare() and the XCMS-based isotopic pair detection that label_compare
    consumes for relative quantification of labeled features
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Produces the grouped peak table (XCMSSet) and supplies sampclass() for labeling
    conditions that label_compare reads
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Execution environment; geoRge is installed via install_github and all steps
    are R calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:5ffa73f1708fe7a626091cf712c490ff11a9b245d93a4816d7faecb090f5bcdc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - geoRge
  - mtbls213
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# isotope-label-fold-change-quantification

## Summary

Runs geoRge's label_compare step on an XCMS feature set to compute fold changes between unlabeled (control) and labeled biologically equivalent samples, producing relative quantification of isotopically enriched LC/MS features in stable isotope-labeling metabolomics.

## When to use

Use when you have an upstream geoRge result (PuInc_seeker + basepeak_finder) plus an XCMS set with control/unlabeled and labeled conditions and need labeled-vs-control fold changes and p-values per feature.

## When NOT to use

- The data set has no unlabeled biologically equivalent control sample to compare against — label_compare depends on comparing isotopic distributions between unlabeled and labeled samples
- The upstream geoRge steps (PuInc_seeker, basepeak_finder) have not been run; label_compare expects the geoRgeR result object as input
- The goal is theoretical isotope-pattern annotation of a single sample rather than empirical labeled-vs-unlabeled comparison — use a different approach (e.g. database_query for adduct/compound annotation, not fold-change quantification)

## Inputs

- XCMS feature set (XCMSSet / xcmsSet) with peak picking, alignment, and grouping already run
- geoRge result object from basepeak_finder (output of PuInc_seeker feeding basepeak_finder)
- Sample-class annotation marking control (unlabeled) vs labeled conditions

## Outputs

- Relative quantification table of labeled features with m/z, retention time, fold change vs control, and p-value columns
- CSV export (e.g. label_compare_results.csv)

## How to apply

Load the mtbls213 XCMSSet (or equivalent) and the geoRge result object produced by basepeak_finder. Assign XCMSSet sample classes with sampclass() distinguishing the control (unlabeled, e.g. CELL_Glc12) from labeled conditions (e.g. CELL_Glc13). Call label_compare(geoRgeR, XCMSet, ppm.s=6.5, control.cond, fc.vs.Control, p.value.vs.Control, Show.bp) so isotopic enrichment shows up as new m/z peaks and higher intensity peaks in the labeled sample relative to the unlabeled one, yielding fold changes across conditions. Write the resulting table (m/z, RT, fold change, p-value) to CSV. The ppm.s=6.5 mass tolerance matches the geoRge default used in the paper's example, and the fc.vs.Control and p.value.vs.Control thresholds should reflect the biological relevance cutoffs chosen for the study (e.g. 1.5-fold, p < 0.05 as used upstream in PuInc_seeker).

## Related tools

- **geoRge** (Provides label_compare() and the XCMS-based isotopic pair detection that label_compare consumes for relative quantification of labeled features) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Produces the grouped peak table (XCMSSet) and supplies sampclass() for labeling conditions that label_compare reads) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Execution environment; geoRge is installed via install_github and all steps are R calls)

## Examples

```
s3 <- label_compare(geoRgeR = s2, XCMSet = mtbls213, ppm.s = 6.5, control.cond = "CELL_Glc12", fc.vs.Control = 1.5, p.value.vs.Control = 0.05)
```

## Evaluation signals

- Output CSV exists and contains labeled-feature rows with fold-change columns for each labeled condition (e.g. normal and high glucose vs control)
- All fold-change values are finite and within a biologically plausible range; the fc.vs.Control threshold from the geoRge call is respected
- p-value columns are populated and pass the p.value.vs.Control threshold used in the call
- Features retained in the output are isotopically enriched pairs consistent with upstream PuInc_seeker / basepeak_finder results (m/z shifted by neutron mass 13.003355-12.0 = 1.003355 per atom, within ppm.s=6.5 tolerance)
- Sample classes in the XCMSSet correctly separate control (unlabeled, e.g. CELL_Glc12) from labeled (e.g. CELL_Glc13) replicates before the comparison

## Limitations

- label_compare relies on the presence of unlabeled biologically equivalent samples; it cannot detect enrichment in datasets where only labeled samples exist
- The method depends entirely on XCMS peak picking, alignment, and grouping parameters — poor feature selection upstream propagates through the fold-change result
- label_compare() had to be patched in geoRge 1.0; older versions may be buggy
- The paper's validation was on LC/MS data from an ARPE-19 cell line grown on normal and high glucose (MTBLS213); other isotope types or acquisition modes may need threshold retuning

## Evidence

- [readme] label_compare is patched in geoRge 1.0: [evidence span withheld — anchor status: external_doc]
