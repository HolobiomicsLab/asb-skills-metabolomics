---
name: george-label-comparison-execution
description: Use when an upstream geoRge PuInc_seeker + basepeak_finder result object
  and an XCMSSet (e.g., MetaboLights MTBLS213) are already available and labeled-vs-unlabeled
  intensity fold changes must be computed per condition.
when_to_use_negative:
- Upstream PuInc_seeker/basepeak_finder steps have not yet been run — label_compare
  expects the PuIncR result object, not raw mzXML files
- The task is isotope annotation against a metabolite database rather than relative
  quantification — use geoRge's database_query with adducts and the reference CSV
  instead
- Only unlabeled samples exist (no labeled condition to compare), so no labeled-vs-control
  fold change can be computed
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: Provides the label_compare function executed in this step; installed from
    GitHub and run on the XCMS output structure
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Supplies the XCMSSet peak table (peak picking, alignment and grouping) and
    sampclass() sample classes that label_compare consumes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: devtools
  role: Used to install geoRge from GitHub (install_github("jcapelladesto/geoRge"))
    before running label_compare
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:fec03740fe02c2af0d65ed9b1405c43a66752295af027e9f84dc7d502351d33c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  - mtbls213
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# george-label-comparison-execution

## Summary

Executes geoRge's label_compare step, which compares XCMS-detected feature intensities between an unlabeled control condition and each isotopically labeled condition (e.g., normal vs. high glucose 13C-labeled samples) to yield relative-quantification fold changes for labeled features.

## When to use

Use when an upstream geoRge PuInc_seeker + basepeak_finder result object and an XCMSSet (e.g., MetaboLights MTBLS213) are already available and labeled-vs-unlabeled intensity fold changes must be computed per condition.

## When NOT to use

- Upstream PuInc_seeker/basepeak_finder steps have not yet been run — label_compare expects the PuIncR result object, not raw mzXML files
- The task is isotope annotation against a metabolite database rather than relative quantification — use geoRge's database_query with adducts and the reference CSV instead
- Only unlabeled samples exist (no labeled condition to compare), so no labeled-vs-control fold change can be computed

## Inputs

- geoRge result object from basepeak_finder (PuIncR)
- XCMS XCMSSet with peak-picked/aligned/grouped features (e.g., mtbls213 from MetaboLights MTBLS213)
- Sample-class assignments separating the unlabeled control condition from labeled conditions (e.g., CELL_Glc12 vs. CELL_Glc13)

## Outputs

- Relative-quantification table of labeled features with m/z, retention time, fold-change (fc.vs.Control), and p-value columns (label_compare_results.csv)
- Labeled-feature rows with finite fold changes for each labeled condition vs. the control

## How to apply

Load the XCMSSet (e.g., mtbls213) and the upstream geoRge result from basepeak_finder as inputs, then assign sample classes with sampclass() so the control (unlabeled) condition is distinguished from each labeled condition. Call label_compare(geoRgeR, XCMSet, ppm.s=6.5, control.cond, fc.vs.Control, p.value.vs.Control, Show.bp), passing the unlabeled control condition name and the labeled conditions to compare (in the MTBLS213 example, CELL_Glc12 unlabeled vs. CELL_Glc13 labeled). label_compare compares isotopic distributions between unlabeled and labeled biologically equivalent samples, so isotopic enrichment manifests as new m/z peaks and higher-intensity peaks in the labeled condition, yielding fold changes rather than iterating over all MS signals with theoretical mass differences. Use the ppm.s=6.5 secondary-isotope matching tolerance consistent with basepeak_finder, and apply the fc.vs.Control threshold (upstream PuInc_seeker used fc.threshold=1.5 and p.value.threshold=0.05) plus PuInc.int.lim/Basepeak.minInt intensity floors (4000/2000) when filtering. Export the resulting labeled-feature table to label_compare_results.csv with m/z, retention time, fold-change, and p-value columns for downstream database_query annotation.

## Related tools

- **geoRge** (Provides the label_compare function executed in this step; installed from GitHub and run on the XCMS output structure) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Supplies the XCMSSet peak table (peak picking, alignment and grouping) and sampclass() sample classes that label_compare consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **devtools** (Used to install geoRge from GitHub (install_github("jcapelladesto/geoRge")) before running label_compare)

## Examples

```
s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",ppm.s=6.5,Basepeak.minInt=2000); res <- label_compare(geoRgeR=s2, XCMSet=mtbls213, ppm.s=6.5, control.cond="CELL_Glc12", fc.vs.Control=1.5, p.value.vs.Control=0.05)
```

## Evaluation signals

- label_compare_results.csv exists and contains labeled-feature rows with fold-change columns for both glucose conditions (normal and high glucose)
- All fold-change values are finite and fall within a plausible range consistent with the applied fc.vs.Control threshold
- Output rows carry m/z, retention time, fold-change, and p-value columns as expected for the relative-quantification table
- Sample-class assignment (sampclass()) correctly separates the unlabeled control condition from the CELL_Glc13 labeled condition before comparison
- ppm.s matches the 6.5 secondary-isotope tolerance used in the upstream basepeak_finder call

## Limitations

- label_compare required patching in geoRge 1.0 ('__`label_compare()`__ has been patched'), so older versions may misbehave
- Depends entirely on XCMS parameters for feature selection, as do competing approaches like X(13)CMS
- Requires labeled and unlabeled biologically equivalent samples; enrichment is detected as new m/z peaks and higher-intensity peaks only when such paired conditions exist

## Evidence

- [abstract] geoRge uses unlabeled and labeled biologically equivalent samples to compare isotopic distributions in the mass spectra: "geoRge uses unlabeled and labeled biologically equivalent samples to compare isotopic distributions in the mass spectra"
- [abstract] Isotopically enriched compounds change their isotopic distribution compared to unlabeled compounds: "Isotopically enriched compounds change their isotopic distribution as compared to unlabeled compounds. This is directly reflected in a number of new m/z peaks and higher intensity peaks in the mass"
- [abstract] geoRge is written in the open language R and runs on the output structure of the XCMS package: "geoRge is written in the open language R and runs on the output structure of the XCMS package"
- [readme] README geoRge 1.0 patch note for label_compare: [evidence span withheld — anchor status: external_doc]
- [readme] README instructions for raising issues: [evidence span withheld — anchor status: external_doc]
