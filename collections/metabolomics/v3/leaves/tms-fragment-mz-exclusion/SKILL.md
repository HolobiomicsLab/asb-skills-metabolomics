---
name: tms-fragment-mz-exclusion
description: Use when preparing GC-EI MS chromatograms (netCDF or mzXML) for eRah
  deconvolution and column bleed or derivatization-reagent fragments (e.g. m/z 73:75,
  147:149, 35:69) would otherwise seed spurious deconvolved compounds.
when_to_use_negative:
- When the m/z range to exclude overlaps analyte-informative fragments required for
  spectral library matching, excluding it would degrade compound identification.
- When data are not GC-MS chromatograms undergoing eRah deconvolution — the exclusion
  is a setDecPar parameter of eRah's CMLC/OSD pipeline, not a general filtering step
  for other peak-picking tools like XCMS or MetAlign.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: R package whose setDecPar() accepts avoid.processing.mz to mask interfering
    fragment ions before deconvolveComp() runs CMLC/OSD deconvolution
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Language in which the exclusion parameter object is constructed and the deconvolution
    is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:6c2ea273aa3657d946260d2d7d496bafee98c948676afdc014e00ca5ff006b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - osd
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tms-fragment-mz-exclusion

## Summary

Excludes fragment ion m/z ranges associated with column bleed, derivatization reagents, and background interference from eRah's spectral deconvolution, using setDecPar(avoid.processing.mz=...) so CMLC/OSD only resolve genuine compound spectra rather than artifacts.

## When to use

Use when preparing GC-EI MS chromatograms (netCDF or mzXML) for eRah deconvolution and column bleed or derivatization-reagent fragments (e.g. m/z 73:75, 147:149, 35:69) would otherwise seed spurious deconvolved compounds.

## When NOT to use

- When the m/z range to exclude overlaps analyte-informative fragments required for spectral library matching, excluding it would degrade compound identification.
- When data are not GC-MS chromatograms undergoing eRah deconvolution — the exclusion is a setDecPar parameter of eRah's CMLC/OSD pipeline, not a general filtering step for other peak-picking tools like XCMS or MetAlign.

## Inputs

- Cleaned GC-MS chromatograms (netCDF or mzXML) loaded as an eRah MetaboSet experiment via newExp()
- List of interfering fragment ion m/z ranges (e.g. c(73:75, 147:149), optionally 35:69) from column bleed or derivatization chemistry

## Outputs

- A Deconvolution parameters object created by setDecPar() carrying avoid.processing.mz exclusions
- Deconvolved MetaboSet whose compound spectra exclude the masked m/z channels (via deconvolveComp())

## How to apply

After loading the cleaned chromatograms into a MetaboSet experiment with newExp(), build the deconvolution parameter object with setDecPar() and pass the interfering fragment ion ranges through the avoid.processing.mz argument. The article and demo configuration exclude m/z 73-75 and 147-149 (the TMS-derivative/background fragments such as 73 from TMS groups and 147 from derivatization artifacts), and optionally extend the exclusion to m/z 35-69 as in the demo setup. The exclusion is applied at the m/z level before deconvolveComp() runs the CMLC filter and OSD blind-source separation, so those ion channels never drive compound detection or contribute to reconstructed pure spectra. Tune the range list to the specific column, reagent chemistry, and instrument background of the experiment; overly broad exclusions risk removing analyte-informative fragments, so verify that remaining spectra still match reference libraries. Combine with min.peak.width, min.peak.height, and noise.threshold settings in the same setDecPar object for a single deconvolution run.

## Related tools

- **erah** (R package whose setDecPar() accepts avoid.processing.mz to mask interfering fragment ions before deconvolveComp() runs CMLC/OSD deconvolution) — http://CRAN.R-project.org/package=erah
- **R** (Language in which the exclusion parameter object is constructed and the deconvolution is executed)

## Examples

```
ex_par <- setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149)); ex <- deconvolveComp(ex, ex_par)
```

## Evaluation signals

- Confirm the setDecPar object was built with avoid.processing.mz including the intended ranges (e.g. c(73:75, 147:149)) before deconvolveComp() is called.
- Check that no deconvolved compound in the output MetaboSet has its leading fragments dominated solely by the excluded m/z channels (73, 74, 75, 147, 148, 149).
- Verify compound spectra of known analytes still achieve high similarity match factors (>80.0) against GMD/MassBank reference libraries after exclusion.
- Benchmark run time and compound yield against the reference configuration (25 samples processed in under 30 minutes on a 2.4 GHz Intel Core i7) to confirm the exclusion did not stall or misconfigure the pipeline.

## Limitations

- The exclusion ranges are instrument/chemistry-specific: eRah does not prescribe universal m/z windows, so the demo values (73:75, 147:149, optionally 35:69) must be validated for the user's column and derivatization reagent.
- Excluding m/z ranges removes those ion channels from downstream compound matching; if a target compound's diagnostic fragments fall inside the excluded window, identification sensitivity is lost.
- eRah exports spectra to MSP format, but the NIST library can only be read with NIST software, so excluded-fragment effects on matching cannot be audited against NIST directly within R.

## Evidence

- [other] Build the deconvolution parameter object with setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149)), optionally extending the exclusion to 35:69 as in the demo configuration.: [evidence span withheld — anchor status: not_found]
- [other] Run deconvolveComp() to apply the CMLC filter and OSD blind-source-separation, resolving co-eluting compounds into pure spectra and computing integrated areas for every sample.: [evidence span withheld — anchor status: not_found]
- [methods] This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [abstract] an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD): "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
