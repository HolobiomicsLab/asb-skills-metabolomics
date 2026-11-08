---
name: irt-peptide-retention-standard-analysis
description: Use when a Thermo Fisher .raw file contains spiked iRT peptides (Biognosys)
  and you need precursor-level XICs or retention-time calibration signals extracted
  directly from the raw file in R.
when_to_use_negative:
- The input is already peak-picked or a feature/transition table (e.g., Skyline or
  OpenMS output) — no raw-file XIC extraction is needed
- The sample does not contain spiked iRT peptides, so no iRT precursor signals are
  expected in the chromatogram window
- The data are not Thermo Fisher raw files — rawrr wraps the vendor RawFileReader
  API and reads only proprietary Thermo .raw data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rawrr
  role: Provides readChromatogram() for direct XIC extraction from Thermo raw files
    with ppm tolerance and scan-type filtering; also ships base R plotting generics
    for the chromatogram set
  repo: https://github.com/fgcz/rawR
- name: tartare
  role: Bioconductor ExperimentData package supplying the example raw file via ExperimentHub
    entry EH4547
- name: ExperimentHub
  role: Used to fetch the example raw file locally (EH4547 <- normalizePath(eh[['EH4547']]))
- name: RawFileReader
  role: Vendor-provided .NET assembly wrapped by rawrr to read spectral and chromatographic
    data from .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:295189b0ba9af70a8444889a16d20906ba67ec3ede52dff853ec52e1b6d26801
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawR
  tools:
  - ExperimentHub
  - R
  - RawFileReader
  - rawR
  - rawrr
  - tartare
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# irt-peptide-retention-standard-analysis

## Summary

Extracts ion chromatograms (XICs) for the 11 Biognosys iRT peptide precursors directly from Thermo raw files using rawrr::readChromatogram with 10 ppm tolerance and MS1-only scans, enabling retention-time standard regression without converting proprietary data.

## When to use

Use when a Thermo Fisher .raw file contains spiked iRT peptides (Biognosys) and you need precursor-level XICs or retention-time calibration signals extracted directly from the raw file in R.

## When NOT to use

- The input is already peak-picked or a feature/transition table (e.g., Skyline or OpenMS output) — no raw-file XIC extraction is needed
- The sample does not contain spiked iRT peptides, so no iRT precursor signals are expected in the chromatogram window
- The data are not Thermo Fisher raw files — rawrr wraps the vendor RawFileReader API and reads only proprietary Thermo .raw data

## Inputs

- Thermo Fisher Scientific Orbitrap raw file (e.g., 20181113_010_autoQC01.raw) containing the iRT peptide mix (Biognosys) in a tryptic BSA digest (NEB)
- iRTmz vector of precursor m/z values for the 11 iRT peptides from the rawrr package's iRT table

## Outputs

- rawrrChromatogramSet containing exactly 11 extracted ion chromatograms (one per iRT precursor m/z)
- Plot of all 11 XICs (e.g., irt_xic.png) using rawrr's base R plotting generics

## How to apply

Obtain the raw file (e.g., 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab) from MassIVE dataset MSV000086542 or via ExperimentHub entry EH4547 from the Bioconductor tartare package. Build the iRTmz vector of precursor m/z values for the 11 iRT peptides (LGGNEQVTR, YILAGVENSK, GTFIIDPGGVIR, GTFIIDPAAVIR, GAGSSEPVTGLDAK, TPVISGGPYEYR, VEATFGVDESNAK, TPVITGAPYEYR, DGLDAASYYAPVR, ADVTPADFSEWSK, LFLQFGAQGSPFLK) from the rawrr package's iRT table. Call rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = 'xic', filter = 'ms') — the 10 ppm tolerance matches precursor mass accuracy and the filter = 'ms' restriction targets MS1-level scans. Verify the returned rawrrChromatogramSet contains exactly 11 chromatograms, then plot them with the base R plotting generics shipped with rawrr. The rationale is that rawrr wraps the vendor RawFileReader .NET assembly, giving complete, OS-independent access to Orbitrap raw data directly in R, keeping the iRT extraction inside the Bioconductor ecosystem for downstream RT regression.

## Related tools

- **rawrr** (Provides readChromatogram() for direct XIC extraction from Thermo raw files with ppm tolerance and scan-type filtering; also ships base R plotting generics for the chromatogram set) — https://github.com/fgcz/rawR
- **tartare** (Bioconductor ExperimentData package supplying the example raw file via ExperimentHub entry EH4547)
- **ExperimentHub** (Used to fetch the example raw file locally (EH4547 <- normalizePath(eh[['EH4547']])))
- **RawFileReader** (Vendor-provided .NET assembly wrapped by rawrr to read spectral and chromatographic data from .raw files) — https://github.com/thermofisherlsms/RawFileReader

## Examples

```
C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = 'xic', filter = 'ms'); plot(C)
```

## Evaluation signals

- The returned object is a rawrrChromatogramSet of length exactly 11, one chromatogram per iRT precursor m/z
- Each of the 11 XICs shows a detectable signal peak eluting within the recorded gradient window (35 min run, 20 min linear C18 gradient at 300 nl/min)
- The 10 ppm tolerance and filter = 'ms' arguments are present in the readChromatogram call, restricting extraction to MS1 scans
- The raw file integrity is confirmed (e.g., MD5 a1f5df9627cf9e0d51ec1906776957ab for 20181113_010_autoQC01.raw)
- The output figure (e.g., irt_xic.png) renders all 11 chromatograms with distinguishable peaks suitable for downstream iRT retention-time regression

## Limitations

- rawrr depends on the vendor RawFileReader .NET assembly, which must be compiled/available (rawrr.exe via dotnet publish) for the target OS (osx/win/linux)
- At manuscript time rawR was not yet in Bioconductor and lacked exchangeable-backend data access and parallel computation support
- XIC quality depends on the assumed precursor charge states in the iRTmz vector; a wrong charge assumption yields no peak within the gradient window

## Evidence

- [results] XIC extraction with 10 ppm mass tolerance using only MS1-level scans (filter = 'ms'): "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [methods] Example raw file obtainable from MassIVE or ExperimentHub: "eh <- ExperimentHub::ExperimentHub()
EH4547 <- normalizePath(eh[["EH4547"]])"
- [abstract] rawR provides complete, OS independent access to Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [methods] Sample composition: iRT peptides spiked into tryptic BSA digest: "The analyzed sample consists of the iRT peptide mix (Biognosys) in a tryptic BSA digest (NEB)"
- [readme] rawrr wraps the vendor RawFileReader .NET assembly; test files come from tartare: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET assembly]"
