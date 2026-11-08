---
name: blind-source-separation-compound-extraction
description: Use when GC/MS chromatograms contain co-eluting compounds and extensive
  EI fragmentation, and pure compound spectra plus integrated areas must be reconstructed
  from raw chromatographic data rather than via peak-picking.
when_to_use_negative:
- The chromatograms have not yet been noise-filtered and baseline-corrected — eRah
  requires the cleaned chromatograms produced by the preceding preprocessing step
  as input.
- Data are already in the form of a fragment-ion peak table or feature matrix (e.g.
  output from XCMS centWave, MetAlign or ChromaTOF) — eRah's quantitative variables
  are compound-spectrum-based, not m/z, retention-time-window and fragment-area based,
  so re-deconvolving a peak table is the wrong scope.
- Only compound identification by spectral library matching is needed and quantification
  is not required — the deconvolution step is specifically the spectrum/area extraction
  stage, downstream of which alignComp(), recMissComp() and identifyComp() operate.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: R package implementing the CMLC + OSD blind source separation deconvolution
    via deconvolveComp() with parameters set by setDecPar(); also provides newExp()
    for loading chromatograms and dataList()/alignList() for extracting results
  repo: https://github.com/xdomingoal/erah-devel
- name: R
  role: Open language in which eRah is written; the deconvolution is invoked as an
    R function chain (newExp → setDecPar → deconvolveComp → save)
- name: GC-TOF MS
  role: Instrument that produced the plasma-sample GC–TOF MS chromatograms used to
    demonstrate the deconvolution workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:89e048a5859a7a73df17a662570e1467bb89cda0be71edd438ebaf9287290b54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xdomingoal/erah-devel
  tools:
  - GC-TOF MS
  - R
  - erah
  - osd
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# blind-source-separation-compound-extraction

## Summary

Applies eRah's two-step blind source separation — compound match by local covariance (CMLC) followed by orthogonal signal deconvolution (OSD) — to extract pure compound spectra and quantitative compound profiles from co-eluted GC–MS chromatograms.

## When to use

Use when GC/MS chromatograms contain co-eluting compounds and extensive EI fragmentation, and pure compound spectra plus integrated areas must be reconstructed from raw chromatographic data rather than via peak-picking.

## When NOT to use

- The chromatograms have not yet been noise-filtered and baseline-corrected — eRah requires the cleaned chromatograms produced by the preceding preprocessing step as input.
- Data are already in the form of a fragment-ion peak table or feature matrix (e.g. output from XCMS centWave, MetAlign or ChromaTOF) — eRah's quantitative variables are compound-spectrum-based, not m/z, retention-time-window and fragment-area based, so re-deconvolving a peak table is the wrong scope.
- Only compound identification by spectral library matching is needed and quantification is not required — the deconvolution step is specifically the spectrum/area extraction stage, downstream of which alignComp(), recMissComp() and identifyComp() operate.

## Inputs

- Cleaned (noise-filtered, baseline-corrected) GC–MS chromatograms in netCDF or mzXML format, loaded into an eRah MetaboSet via newExp() with an instrumental and a phenotype table
- Deconvolution parameter object created with setDecPar (min.peak.width, min.peak.height, noise.threshold, avoid.processing.mz)

## Outputs

- MetaboSet object containing deconvolved pure compound spectra per sample
- Integrated area / quantitative compound profile per resolved compound and sample, exportable via dataList(ex)

## How to apply

Load cleaned chromatograms (netCDF or mzXML) into an eRah MetaboSet experiment with newExp(), then build a deconvolution parameter object with setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149)) before running deconvolveComp(). deconvolveComp() first applies CMLC to detect compounds by exploiting ion redundancy covariance patterns, then uses OSD (a blind source separation technique) to retrieve the pure compound spectrum given the elution profile. The resulting pure spectrum is fed to a LAD (least absolute deviation) regression that yields the quantitative compound profile (integrated area) for every sample. Optionally register a parallel backend (e.g. plan(future::multisession, workers=2)) because eRah was benchmarked processing 25 GC–TOF MS plasma samples in under 30 minutes on a 2.4 GHz Intel Core i7. Save the resolved experiment (e.g. save(ex, file='deconvolved_experiment.rda')) and confirm the MetaboSet contains deconvolved spectra and areas for each sample before downstream alignment and identification steps.

## Related tools

- **erah** (R package implementing the CMLC + OSD blind source separation deconvolution via deconvolveComp() with parameters set by setDecPar(); also provides newExp() for loading chromatograms and dataList()/alignList() for extracting results) — https://github.com/xdomingoal/erah-devel
- **R** (Open language in which eRah is written; the deconvolution is invoked as an R function chain (newExp → setDecPar → deconvolveComp → save))
- **GC-TOF MS** (Instrument that produced the plasma-sample GC–TOF MS chromatograms used to demonstrate the deconvolution workflow)

## Examples

```
ex <- newExp(...); dec.par <- setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75,147:149)); ex <- deconvolveComp(ex, dec.par)
```

## Evaluation signals

- The returned MetaboSet contains deconvolved spectra and areas for every sample — verify by inspecting the object after save(ex, file='deconvolved_experiment.rda') or via alignList(ex)/dataList(ex).
- Resolved compounds match reference MS spectra with a high similarity match factor (>80.0) in libraries such as GMD or MassBank, confirming the pure spectra are chemically meaningful.
- Processing time is consistent with the article benchmark — 25 samples in under 30 minutes on a 2.4 GHz Intel Core i7; substantially longer runtimes suggest mis-set setDecPar thresholds (e.g. noise.threshold or min.peak.height too low).
- Spectra found in the experiment can be exported to MSP format via eRah's export function and remain readable outside NIST software, confirming the deconvolved spectra are in a non-binary readable format.
- Quantified areas from deconvolveComp() should be comparable to (not confounded by) peak-picking outputs from XCMS centWave, MetAlign or ChromaTOF — differences should reflect co-elution resolution rather than missing signals.

## Limitations

- eRah's quantitative variables are compound-spectrum based (m/z value, retention time window, area of fragment ion peaks are not the primary output), so results are not directly interchangeable with peak-picking tables.
- NIST library spectra can only be read with NIST software, so deconvolved spectra must be exported to MSP format or matched against libraries like GMD or MassBank for identification.
- Avoiding specific ion traces (e.g. m/z 73:75, 147:149, optionally 35:69 as in the demo configuration) is required because reagent ions and column bleed dominate covariance and distort CMLC detection.
- The development version is distributed via GitHub while the CRAN release is the recommended download — development builds may be less stable and bug reports should include a reproducible example at the erah-devel issues page.

## Evidence

- [abstract] an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD): "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
- [intro] Compounds are quantified and identified on the basis of a multivariate deconvolution process22 that extracts and constructs pure compound spectra from raw data: [evidence span withheld — anchor status: not_found]
- [methods] The compounds in data are deconvolved using `deconvolveComp()` ... This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function: "The compounds in data are deconvolved using `deconvolveComp()`"
- [results] These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank.: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [readme] Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics.: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
