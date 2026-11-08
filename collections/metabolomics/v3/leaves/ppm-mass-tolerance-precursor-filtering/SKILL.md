---
name: ppm-mass-tolerance-precursor-filtering
description: Use when you need chromatograms for known precursor m/z values (e.g.,
  iRT peptides) directly from a Thermo Fisher .raw file with a ppm mass tolerance,
  without converting to mzML or using vendor GUI software.
when_to_use_negative:
- When extracting fragment-ion signal from MS2 scans — use the scan-filter form of
  readChromatogram with a filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00
  [100.0000-1015.0000]' instead of the MS1-only 'ms' filter.
- When the input is already converted/centroided open-format data (mzML) or a feature/quantification
  table — no raw-file access is needed.
- When the data are not from Thermo Fisher Scientific instruments, since rawrr wraps
  the vendor-provided RawFileReader API for Thermo .raw files only.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rawR
  role: Provides rawrr::readChromatogram for direct, OS-independent XIC extraction
    from Thermo .raw files with ppm tolerance and scan filters
  repo: https://github.com/fgcz/rawR
- name: RawFileReader
  role: Vendor-provided .NET assembly wrapped by rawrr that performs the actual raw-file
    reading
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: tartare (ExperimentHub)
  role: Bioconductor ExperimentData package supplying the example raw file via ExperimentHub
    entry EH4547
  repo: https://bioconductor.org/packages/tartare/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:fd4219d453a88813bd19088d8dea4aeba95c39bdfca695f1ad61a51f75019961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawR
  tools:
  - ExperimentHub
  - FIDDLE
  - R
  - RawFileReader
  - rawR
  - tartare
  - tartare (ExperimentHub)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- precursor-ppm-mass-tolerance-filtering
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ppm-mass-tolerance-precursor-filtering

## Summary

Extract precursor-level chromatographic signal from Thermo raw files by computing extracted ion chromatograms (XICs) around expected precursor m/z values with a ppm-based mass tolerance, restricted to MS1-level scans via rawrr::readChromatogram.

## When to use

Use when you need chromatograms for known precursor m/z values (e.g., iRT peptides) directly from a Thermo Fisher .raw file with a ppm mass tolerance, without converting to mzML or using vendor GUI software.

## When NOT to use

- When extracting fragment-ion signal from MS2 scans — use the scan-filter form of readChromatogram with a filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' instead of the MS1-only 'ms' filter.
- When the input is already converted/centroided open-format data (mzML) or a feature/quantification table — no raw-file access is needed.
- When the data are not from Thermo Fisher Scientific instruments, since rawrr wraps the vendor-provided RawFileReader API for Thermo .raw files only.

## Inputs

- Thermo Fisher Scientific raw file (e.g., 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab from MassIVE MSV000086542 or ExperimentHub entry EH4547 from the tartare package)
- Numeric vector of target precursor m/z values (e.g., iRTmz built from the rawrr iRT peptide table)

## Outputs

- rawrrChromatogramSet of extracted ion chromatograms (one per input m/z, e.g., 11 XICs for the iRT peptides)
- Plotted figure of all XICs (e.g., irt_xic.png) using rawrr's base R plotting generics

## How to apply

Build the target precursor m/z vector (e.g., iRTmz for the 11 iRT peptides: LGGNEQVTR, YILAGVENSK, GTFIIDPGGVIR, GTFIIDPAAVIR, GAGSSEPVTGLDAK, TPVISGGPYEYR, VEATFGVDESNAK, TPVITGAPYEYR, DGLDAASYYAPVR, ADVTPADFSEWSK, LFLQFGAQGSPFLK) from the rawrr package's iRT table, using charge-state-adjusted m/z values. Call rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = 'xic', filter = 'ms') — tol = 10 specifies a 10 ppm mass window and filter = 'ms' restricts extraction to MS1-level scans, which is correct for precursor (survey-scan) signal. The function wraps Thermo's RawFileReader .NET assembly, so it reads the proprietary .raw file directly and is OS-independent (Windows/Linux/macOS). Plot the returned rawrrChromatogramSet with the base R plotting generics shipped with rawrr to inspect peak elution within the recorded gradient window (e.g., the 35 min, 20 min gradient run of 20181113_010_autoQC01.raw).

## Related tools

- **rawR** (Provides rawrr::readChromatogram for direct, OS-independent XIC extraction from Thermo .raw files with ppm tolerance and scan filters) — https://github.com/fgcz/rawR
- **RawFileReader** (Vendor-provided .NET assembly wrapped by rawrr that performs the actual raw-file reading) — https://github.com/thermofisherlsms/RawFileReader
- **tartare (ExperimentHub)** (Bioconductor ExperimentData package supplying the example raw file via ExperimentHub entry EH4547) — https://bioconductor.org/packages/tartare/

## Examples

```
C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms"); plot(C)
```

## Evaluation signals

- The returned object is a rawrrChromatogramSet containing exactly 11 chromatograms, one per input m/z value.
- Each XIC shows a detectable signal peak eluting within the recorded gradient window (the 35 min run; 20 min linear C18 gradient at 300 nl/min).
- Mass tolerance was applied in ppm (tol = 10), not Da, so the extraction window scales with the absolute m/z of each precursor.
- The extraction used only MS1-level scans (filter = 'ms'); no MS2-derived signal appears in the precursor XICs.
- TIC calculated with readChromatogram(type = 'tic') from the same MS1 scans shows a chromatogram consistent with the XIC elution order (iRT peptides are indexed by retention time).

## Limitations

- rawrr requires the RawFileReader .NET assembly and its compiled rawrr.exe (built via 'dotnet publish rawrr-dotnet.csproj --os osx -a x64 ...'), so a .NET runtime must be installed on Windows, Linux, or macOS.
- At the article's time, rawR was not yet in Bioconductor and lacked exchangeable-backend and parallel-computation support.
- The 10 ppm tolerance and MS1-only filter are manuscript-specific settings; other applications (e.g., fragment-ion XICs from MS2 scans) require different filter strings and tolerance choices.

## Evidence

- [results] XIC extraction with 10 ppm mass tolerance using only MS1-level scans: "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [results] Computing XICs for iRT peptide precursors: "this is achieved by computing an extracted ion chromatogram (XIC) for iRT peptide precursors, provided they were spiked into the sample matrix"
- [abstract] rawR provides OS-independent access to Thermo raw files: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [readme] rawrr wraps the vendor RawFileReader .NET assembly and test files come from tartare: [evidence span withheld — anchor status: not_found]
- [methods] Input raw file available via ExperimentHub EH4547: "eh <- ExperimentHub::ExperimentHub()
EH4547 <- normalizePath(eh[["EH4547"]])"
