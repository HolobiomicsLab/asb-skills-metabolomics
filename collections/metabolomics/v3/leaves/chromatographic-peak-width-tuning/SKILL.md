---
name: chromatographic-peak-width-tuning
description: Use when configuring eRah deconvolveComp() on GC-EI MS chromatograms
  (netCDF/mzXML) and you must set min.peak.width, min.peak.height, noise.threshold,
  or avoid.processing.mz before running CMLC/OSD.
when_to_use_negative:
- Input is already a deconvolved MetaboSet and the task is downstream alignment or
  identification (use setAlPar/alignComp or identifyComp instead)
- Data are LC-MS or non-EI where ion-redundancy covariance patterns exploited by CMLC/OSD
  do not apply
- The goal is generic peak picking of individual fragment ions (e.g. XCMS centWave)
  rather than resolving co-eluting compounds into pure spectra
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: erah
  role: R package whose setDecPar()/deconvolveComp() implement the parameterized CMLC
    and OSD deconvolution being tuned
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which the deconvolution parameter object is constructed and
    the pipeline is executed
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:ce5b19fec8a0af3e0fd99c6a2eea8ab2787f7bffc77883187376c386cf400564
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - osd
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# chromatographic-peak-width-tuning

## Summary

Tunes eRah's deconvolution parameter object (setDecPar) — minimum peak width, minimum peak height, noise threshold, and excluded m/z ranges — so that CMLC/OSD spectral deconvolution resolves co-eluted GC-MS compounds without fragmenting or inflating peaks.

## When to use

Use when configuring eRah deconvolveComp() on GC-EI MS chromatograms (netCDF/mzXML) and you must set min.peak.width, min.peak.height, noise.threshold, or avoid.processing.mz before running CMLC/OSD.

## When NOT to use

- Input is already a deconvolved MetaboSet and the task is downstream alignment or identification (use setAlPar/alignComp or identifyComp instead)
- Data are LC-MS or non-EI where ion-redundancy covariance patterns exploited by CMLC/OSD do not apply
- The goal is generic peak picking of individual fragment ions (e.g. XCMS centWave) rather than resolving co-eluting compounds into pure spectra

## Inputs

- Cleaned GC-MS chromatograms (netCDF or mzXML) loaded into an eRah MetaboSet via newExp()
- Instrumental and phenotype metadata tables
- Candidate threshold values for min.peak.width, min.peak.height, noise.threshold, and avoid.processing.mz

## Outputs

- A setDecPar Deconvolution parameters object consumed by deconvolveComp()
- Deconvolved MetaboSet containing pure compound spectra and integrated areas per sample

## How to apply

Build a Deconvolution parameters object with setDecPar() before calling deconvolveComp(), since deconvolution requires these parameters to be set a priori. The workflow card demonstrates concrete starting values: min.peak.width=1, min.peak.height=2000, and noise.threshold=500, with avoid.processing.mz=c(73:75, 147:149) to exclude column-bleed/siloxane ions, optionally extending the exclusion to m/z 35:69 as in the demo configuration. Peak-width and height thresholds control what CMLC accepts as a compound-defining chromatographic peak, while the noise threshold gates low-abundance signal that would otherwise seed spurious deconvolution. For larger sample sets (the article's benchmark processed 25 samples in under 30 minutes on a 2.4 GHz Intel Core i7), register a parallel backend such as plan(future::multisession, workers=2) before deconvolution. Tune thresholds iteratively: too-lenient width/height admit noise-driven artifacts, too-strict values drop genuine low-abundance metabolites, so inspect resolved spectra and integrated areas after each run.

## Related tools

- **erah** (R package whose setDecPar()/deconvolveComp() implement the parameterized CMLC and OSD deconvolution being tuned) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which the deconvolution parameter object is constructed and the pipeline is executed) — http://CRAN.R-project.org/package=erah

## Examples

```
dec.par <- setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149)); ex <- deconvolveComp(ex, dec.par)
```

## Evaluation signals

- The saved MetaboSet (save(ex, file='deconvolved_experiment.rda')) contains deconvolved spectra and integrated areas for every sample
- Column-bleed ions (m/z 73:75, 147:149, optionally 35:69) are absent from resolved compound spectra
- No spurious low-intensity compounds survive when the noise.threshold=500 gate is respected; raising min.peak.height=2000 suppresses noise-seeded peaks
- Runtime scales sensibly: 25 samples deconvolve in under 30 minutes on a 2.4 GHz Intel Core i7 (2-worker multisession backend)

## Limitations

- eRah was validated on GC–TOF MS serum data; thresholds may not transfer to other instrument platforms
- No specific version numbers stated for eRah in the article, so exact parameter defaults may differ across package versions
- Excluding broad m/z ranges (e.g. 35:69) risks suppressing genuine low-mass fragments of real metabolites

## Evidence

- [methods] deconvolution parameters: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [methods] deconvolveComp: "The compounds in data are deconvolved using `deconvolveComp()`"
- [abstract] CMLC and OSD: "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
- [abstract] noise and baseline: "(i) noise filtering and baseline removal of GC/MS chromatograms"
- [readme] installation from README: "install.packages('erah')

To install this development version, execute:

remotes::install_github('xdomingoal/erah-devel')"
