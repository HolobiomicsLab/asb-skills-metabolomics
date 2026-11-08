---
name: extracted-ion-chromatogram-extraction
description: Use when you hold a Thermo .raw file (e.g. from MassIVE MSV000086542
  or ExperimentHub EH4547) and need XIC signal traces around known precursor m/z values
  — such as spiked-in iRT peptides — without converting to mzML.
when_to_use_negative:
- The data is already in an open format (mzML/mzXML) or already a feature/peak table
  — the skill is specific to direct access to proprietary Thermo raw files via RawFileReader.
- You need total ion chromatograms or method-optimization visualizations rather than
  m/z-targeted XICs — rawDiag's ggplot2-based QC plots are the more appropriate tool.
- The instrument is not Thermo Fisher Scientific; rawrr wraps the vendor-provided
  RawFileReader .NET assembly and only reads Thermo .raw files.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: rawR/rawrr
  role: Provides readChromatogram() to compute XICs directly from Thermo raw files
    with ppm tolerance and scan filters; ships base R plotting generics for the returned
    rawrrChromatogramSet
  repo: https://github.com/fgcz/rawR
- name: tartare
  role: Bioconductor ExperimentData package supplying the example raw file through
    ExperimentHub entry EH4547
  repo: https://bioconductor.org/packages/tartare/
- name: RawFileReader
  role: Vendor-provided .NET assembly wrapped by rawrr that performs the actual reading
    of chromatographic and spectral data from the .raw file
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:da2f6e0c27670a394e6c853cea6be12f6d210c29b8063d0e64ef4f7596375d94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawR
  tools:
  - ExperimentHub
  - R
  - RawFileReader
  - rawR
  - rawR/rawrr
  - tartare
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# extracted-ion-chromatogram-extraction

## Summary

Generates extracted ion chromatograms (XICs) directly from Thermo Fisher Scientific Orbitrap .raw files using rawrr::readChromatogram, e.g. for the 11 iRT peptide precursors in 20181113_010_autoQC01.raw with a 10 ppm tolerance restricted to MS1-level scans, producing a rawrrChromatogramSet for plotting and downstream retention-time work.

## When to use

Use when you hold a Thermo .raw file (e.g. from MassIVE MSV000086542 or ExperimentHub EH4547) and need XIC signal traces around known precursor m/z values — such as spiked-in iRT peptides — without converting to mzML.

## When NOT to use

- The data is already in an open format (mzML/mzXML) or already a feature/peak table — the skill is specific to direct access to proprietary Thermo raw files via RawFileReader.
- You need total ion chromatograms or method-optimization visualizations rather than m/z-targeted XICs — rawDiag's ggplot2-based QC plots are the more appropriate tool.
- The instrument is not Thermo Fisher Scientific; rawrr wraps the vendor-provided RawFileReader .NET assembly and only reads Thermo .raw files.

## Inputs

- Thermo Fisher Scientific .raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- Vector of target precursor m/z values (e.g. iRTmz for the 11 iRT peptides from the rawrr iRT table)
- Mass tolerance in ppm (10 ppm) and scan filter string ('ms' for MS1, or an explicit FTMS ms2 filter)

## Outputs

- rawrrChromatogramSet of 11 extracted ion chromatograms (one per target m/z)
- Plotted XIC figure (e.g. irt_xic.png) via rawrr's base R plotting generics

## How to apply

Obtain the raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab, from MassIVE MSV000086542 or via ExperimentHub entry EH4547 from the Bioconductor tartare package). Build the precursor m/z vector (iRTmz) for the target peptides — e.g. the 11 iRT peptides LGGNEQVTR through LFLQFGAQGSPFLK from the rawrr package's iRT table. Call rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = 'xic', filter = 'ms') where tol = 10 sets a 10 ppm mass tolerance and filter = 'ms' restricts extraction to MS1-level scans; for fragment ions within a specific MS2 event, supply an explicit filter string such as 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'. Plot the returned rawrrChromatogramSet with the base R plotting generics shipped with rawrr to minimize dependencies. Validate that the set contains exactly the expected number of chromatograms (11 for the full iRT mix), each with a detectable peak eluting within the recorded gradient window (a 35 min run here with 2.1881 × 10^4 scans).

## Related tools

- **rawR/rawrr** (Provides readChromatogram() to compute XICs directly from Thermo raw files with ppm tolerance and scan filters; ships base R plotting generics for the returned rawrrChromatogramSet) — https://github.com/fgcz/rawR
- **tartare** (Bioconductor ExperimentData package supplying the example raw file through ExperimentHub entry EH4547) — https://bioconductor.org/packages/tartare/
- **RawFileReader** (Vendor-provided .NET assembly wrapped by rawrr that performs the actual reading of chromatographic and spectral data from the .raw file) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = 'xic', filter = 'ms'); plot(C)
```

## Evaluation signals

- The returned object is a rawrrChromatogramSet containing exactly 11 chromatograms when all 11 iRT precursor m/z values are queried.
- Each chromatogram shows a detectable signal peak eluting within the recorded gradient window (the 35 min run).
- Input file integrity is confirmed via MD5 a1f5df9627cf9e0d51ec1906776957ab for 20181113_010_autoQC01.raw.
- The 10 ppm tolerance and filter = 'ms' settings yield XICs whose apex retention times are consistent enough to support downstream iRT retention-time regression.
- XICs can be cross-checked against the MS1 total ion chromatogram (readChromatogram(type = 'tic')) computed from the same raw file.

## Limitations

- rawrr (at manuscript time) had no parallel computation support and no exchangeable backend support for data access; both were listed as planned future work.
- Extraction is limited to Thermo Fisher Scientific raw files, since rawrr wraps the vendor-provided RawFileReader API.
- Fragment-ion XICs require specifying an exact scan filter string (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'), so the filter must match the instrument method's isolation m/z and HCD energy.

## Evidence

- [results] XIC extraction with 10 ppm mass tolerance using only MS1-level scans: "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [methods] Example raw file provenance via ExperimentHub: "eh <- ExperimentHub::ExperimentHub()
EH4547 <- normalizePath(eh[["EH4547"]])"
- [abstract] rawR provides OS-independent access to Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [results] XICs computed for iRT peptide precursors spiked into the sample: "this is achieved by computing an extracted ion chromatogram (XIC) for iRT peptide precursors, provided they were spiked into the sample matrix"
- [readme] rawrr wraps the vendor RawFileReader assembly; test data from tartare: [evidence span withheld — anchor status: not_found]
