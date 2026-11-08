---
name: ms2-compound-annotation-filtering
description: Use when untargeted LC-MS/MS data (DDA or DIA, with MS2 spectra from
  pooled QC samples) needs compound-level annotation with fragment-level evidence,
  and noisy/blank-contaminated features must be filtered before annotation.
when_to_use_negative:
- The input is already an annotated compound concentration table — annotation against
  MS2 reference databases is unnecessary; proceed to statistical or pathway analysis.
- The data are MS1-only (no tandem MS spectra collected), so fragment-level MS2 annotation
  cannot be supported.
- The goal is targeted quantification of a predefined compound panel rather than untargeted
  discovery annotation.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnalystR 4.0
  role: R package executing the MS2 spectra deconvolution, compound annotation, and
    feature-filtering functions (e.g. FilterVariable, CentroidCheck) underlying the
    MetaboAnalyst web server
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform hosting the MS2 Peak Annotation module with comprehensive MS2
    reference databases and fragment-level annotation
- name: asari
  role: Algorithm integrated into the enhanced Spectra Processing module for LC-MS1
    spectra processing upstream of MS2 annotation
- name: OptiLCMS
  role: Companion package for LC-MS feature detection installed alongside MetaboAnalystR
  repo: https://github.com/xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:e4ed07c3cfd9c2b4de1aeea313a198705d7a843d1636bef2229d763498d8633b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  - asari
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- compound-annotation-redundancy-filtering
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ms2-compound-annotation-filtering

## Summary

MetaboAnalyst 6.0's MS2 Peak Annotation workflow: annotating LC-MS/MS features at fragment level against comprehensive MS2 reference databases (~1.5 million spectra), with pre-annotation feature filtering (QC-based RSD filtering, blank subtraction, centroid checking) to control false identifications in untargeted metabolomics.

## When to use

Use when untargeted LC-MS/MS data (DDA or DIA, with MS2 spectra from pooled QC samples) needs compound-level annotation with fragment-level evidence, and noisy/blank-contaminated features must be filtered before annotation.

## When NOT to use

- The input is already an annotated compound concentration table — annotation against MS2 reference databases is unnecessary; proceed to statistical or pathway analysis.
- The data are MS1-only (no tandem MS spectra collected), so fragment-level MS2 annotation cannot be supported.
- The goal is targeted quantification of a predefined compound panel rather than untargeted discovery annotation.

## Inputs

- Raw or centroided LC-MS/MS spectra files (MS1 and MS2; DDA or DIA acquisition)
- Pooled QC-sample MS2 spectra collected to assist compound identification
- MS1 peak/feature tables with m/z, retention time, and sample intensities
- Blank (procedural control) samples for blank subtraction

## Outputs

- MS2-based compound annotations with fragment-level annotation for each feature
- Filtered feature table (QC-passing, blank-subtracted, variance/intensity-filtered features)
- Annotated compound list attachable to an mSet object (e.g. via Convert2Mummichog) for functional analysis

## How to apply

Start from raw or centroided LC-MS spectra; remove empty MS scans and verify centroiding (CentroidCheck), then apply FilterVariable with QC filtering (qcFilter, rsd threshold), variance/intensity cutoffs, and blank.subtraction with a blank.threshold to remove features attributable to carryover or background. Deconvolve DIA MS2 data to reconstruct precursor-ion associations, or use DDA MS2 directly, then match MS2 spectra against MetaboAnalyst's spectra databases (~1.5 million MS2 spectra) to obtain fragment-level compound annotations. The rationale is that incorporating MS2 processing and annotation into untargeted workflows 'greatly improve[s] compound annotations and functional interpretation', and benchmarking showed the MetaboAnalystR 4.0 pipeline increases the true-positive chemical identification rate by >40% for DDA and DIA datasets without increasing false identifications. Downstream, annotated compounds can be fed into mummichog-style functional analysis via Convert2Mummichog.

## Related tools

- **MetaboAnalystR 4.0** (R package executing the MS2 spectra deconvolution, compound annotation, and feature-filtering functions (e.g. FilterVariable, CentroidCheck) underlying the MetaboAnalyst web server) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform hosting the MS2 Peak Annotation module with comprehensive MS2 reference databases and fragment-level annotation)
- **asari** (Algorithm integrated into the enhanced Spectra Processing module for LC-MS1 spectra processing upstream of MS2 annotation)
- **OptiLCMS** (Companion package for LC-MS feature detection installed alongside MetaboAnalystR) — https://github.com/xia-lab/OptiLCMS

## Examples

```
install.packages("devtools"); library(devtools); devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Every retained feature carries a fragment-level annotation traceable to a match in the MS2 reference databases (~1.5 million spectra).
- QC filtering is verifiable: features failing the qcFilter/RSD criterion or blank.threshold in blank.subtraction are removed before annotation.
- Benchmark expectation: for DDA and DIA datasets the true-positive chemical identification rate increases by >40% without an increase in false identifications.
- Serial-dilution check: MetaboAnalystR 4.0 should detect and identify >10% more high-quality MS and MS/MS features than the prior pipeline.
- CentroidCheck confirms data are centroided and empty MS scans are removed before downstream annotation.

## Limitations

- DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions; un-deconvoluted DIA spectra cannot be matched directly.
- MS2 spectra are typically only collected from pooled QC samples, so compound identification depends on pooled-quality spectra rather than per-sample spectra.
- As stated in the article's discussion, metabolome annotations still need improvement and remain future work.

## Evidence

- [abstract] a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation: "a MS2 Peak Annotation module based on comprehensive MS2 reference databases with fragment-level annotation"
- [intro] To assist with compound identification, tandem MS spectra are often collected from pooled QC samples: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [methods] FilterVariable function signature with qcFilter, rsd, blank.subtraction, blank.threshold: "`FilterVariable(mSetObj, filter, qcFilter, rsd, qc.filter, var.filter, var.cutoff, int.filter, int.cutoff, blank.subtraction, blank.threshold)`"
- [readme] README benchmark claim on DDA/DIA true-positive improvement: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
- [readme] README description of the MS/MS deconvolution and annotation module: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
