---
name: pooled-qc-spectral-quality-check
description: Use when processing untargeted LC-MS/MS raw spectral data (e.g. mzML/mzData)
  that includes pooled-QC MS2 acquisitions — especially SWATH-DIA runs — and you need
  to verify MS2 spectral quality and assign compound identities.
when_to_use_negative:
- Input is already a processed feature table — this skill operates on raw MS/MS spectra
  and pooled-QC acquisitions, not on quantified peak tables
- The dataset contains no pooled-QC (or other) MS2 spectra, so no spectral quality
  check or MS2-based identification is possible
- Targeted metabolomics where identities and transitions are predefined and MS2 database
  deconvolution adds no value
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: R implementation of the MS/MS spectra deconvolution and compound annotation
    module for DDA and DIA data, including the ~1.5M MS2 spectra database
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Underlying LC-MS1 feature detection engine used within the MetaboAnalystR
    pipeline before MS2 annotation
  repo: https://github.com/xia-lab/OptiLCMS
- name: MetaboAnalyst 6.0
  role: Web platform hosting the enhanced Spectra Processing module with MS2 support
    and the MS2 Peak Annotation module
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:f8b3a8ad998bf24136cac57eb4af1c35e4203b3160227377034396f1cb581919
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pooled-qc-spectral-quality-check

## Summary

Leverages tandem MS (MS2) spectra acquired on pooled QC samples to check spectral quality and support compound identification in untargeted LC-MS metabolomics, using MetaboAnalyst 6.0/MetaboAnalystR 4.0's MS/MS deconvolution and annotation module against a ~1.5 million MS2 spectra database.

## When to use

Use when processing untargeted LC-MS/MS raw spectral data (e.g. mzML/mzData) that includes pooled-QC MS2 acquisitions — especially SWATH-DIA runs — and you need to verify MS2 spectral quality and assign compound identities.

## When NOT to use

- Input is already a processed feature table — this skill operates on raw MS/MS spectra and pooled-QC acquisitions, not on quantified peak tables
- The dataset contains no pooled-QC (or other) MS2 spectra, so no spectral quality check or MS2-based identification is possible
- Targeted metabolomics where identities and transitions are predefined and MS2 database deconvolution adds no value

## Inputs

- Raw LC-MS/MS2 spectral files (e.g. the islet high/low-glucose samples plus pooled-QC SWATH-DIA acquisitions)
- Pooled-QC MS2 spectra (DDA or DIA/SWATH format)
- MS2 reference spectra database (~1.5 million MS2 spectra bundled with MetaboAnalystR 4.0)

## Outputs

- Deconvoluted MS2 spectra with reconstructed fragment-to-precursor associations
- MS2 peak annotation results with fragment-level annotation and compound identity assignments
- Pooled-QC spectral quality summary (centroid check, empty-scan removal, signal drift correction status)

## How to apply

First confirm the experiment collected tandem MS spectra on pooled QC samples, since these QC MS2 runs are what assist compound identification. For DIA (e.g. SWATH) data, run the MS/MS spectra deconvolution step in MetaboAnalyst 6.0's Spectra Processing module (or MetaboAnalystR 4.0, which uses OptiLCMS for LC-MS1 peak detection) to reconstruct associations between fragment ions and their precursor ions. Check the deconvoluted spectra for quality (centroid status via CentroidCheck, empty MS scans removed, signal drift corrected) before annotation. Then annotate the deconvoluted MS2 features against the ~1.5 million MS2 reference spectra knowledgebase to assign named compound identities, relying on fragment-level annotation. The rationale is that MS2 processing and annotation greatly improves compound annotations relative to MS1-only processing, and pooled-QC MS2 provides the broadest, most reproducible spectral coverage of the study's features.

## Related tools

- **MetaboAnalystR 4.0** (R implementation of the MS/MS spectra deconvolution and compound annotation module for DDA and DIA data, including the ~1.5M MS2 spectra database) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (Underlying LC-MS1 feature detection engine used within the MetaboAnalystR pipeline before MS2 annotation) — https://github.com/xia-lab/OptiLCMS
- **MetaboAnalyst 6.0** (Web platform hosting the enhanced Spectra Processing module with MS2 support and the MS2 Peak Annotation module)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Deconvolution successfully reconstructs fragment-to-precursor associations for DIA (SWATH) MS2 spectra without orphaned fragments
- Annotated MS2 features receive at least one potential named chemical identity via the ~1.5M spectra reference database
- For the human islet case study, the regenerated pipeline should approach the reported benchmark of 27,209 MS1 features with 4,959 features carrying named identities
- CentroidCheck and empty-MS-scan removal confirm spectra are in analyzable centroided form before deconvolution
- MetaboAnalystR benchmark expectation: increased true-positive rate of chemical identification (>40% reported) without increased false identifications

## Limitations

- DIA MS2 data require explicit deconvolution to reconstruct precursor associations; skipping this step invalidates downstream annotation
- Annotation coverage depends on the reference spectra database; the authors note metabolome annotations still need improvement (future work)
- Benchmark gains (>10% more high-quality features, >40% TPR) are reported for the authors' benchmark studies and may not transfer to all datasets

## Evidence

- [intro] pooled QC provides MS2 for identification: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [intro] DIA needs deconvolution: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [methods] MS2 reference database size: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] README deconvolution module scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
