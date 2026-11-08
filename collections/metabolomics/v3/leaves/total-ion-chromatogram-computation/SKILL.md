---
name: total-ion-chromatogram-computation
description: Use when starting analysis of an unconverted Thermo Fisher raw file and
  you need a first impression of the LC-MS run (e.g. instrument QC of an autoQC sample)
  as a total ion chromatogram spanning all MS1-level scans.
when_to_use_negative:
- The data are already in an exchange format such as mzML — use a converter-based
  reader instead of the vendor-API wrapper
- You need extracted ion chromatograms for specific fragment ions within an MS2 scan
  — use a scan filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'
  rather than a plain TIC
- You require rawDiag-style LC-MS method optimization diagnostics beyond a plain TIC
  overview
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: rawrr
  role: R package whose readChromatogram(type = 'tic') function drives rawrr.exe to
    compute and return the TIC as a rawrrChromatogram object
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Thermo Fisher .NET (C#) assemblies that rawrr wraps; the vendor API performs
    scan filtering and provides chromatographic data from the RAW file
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: rawR
  role: Manuscript predecessor providing OS-independent access to all spectral data
    logged in Thermo Fisher Scientific raw files
  repo: https://github.com/fgcz/rawR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:f8700370492a0b3aefde63db04851e4a89b845ff9d931b76e47da66330c78808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - .NET
  - Bioconductor
  - Mono
  - R
  - RawFileReader
  - rawR
  - rawrr
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# total-ion-chromatogram-computation

## Summary

Computes a total ion chromatogram (TIC) directly from a proprietary Thermo Fisher Scientific .raw file using the rawrr package, which delegates scan filtering and MS1-level aggregation to the vendor RawFileReader API via the rawrr.exe C# assembly rather than converting to mzML first.

## When to use

Use when starting analysis of an unconverted Thermo Fisher raw file and you need a first impression of the LC-MS run (e.g. instrument QC of an autoQC sample) as a total ion chromatogram spanning all MS1-level scans.

## When NOT to use

- The data are already in an exchange format such as mzML — use a converter-based reader instead of the vendor-API wrapper
- You need extracted ion chromatograms for specific fragment ions within an MS2 scan — use a scan filter string like 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]' rather than a plain TIC
- You require rawDiag-style LC-MS method optimization diagnostics beyond a plain TIC overview

## Inputs

- Proprietary Thermo Fisher Scientific raw file (e.g. 20181113_010_autoQC01.raw, MD5 a1f5df9627cf9e0d51ec1906776957ab)
- rawrr.exe assembly with ThermoFisher.CommonCore RawFileReader .NET assemblies and a working .NET 8.0 or Mono runtime

## Outputs

- rawrrChromatogram S3 object containing the TIC computed over all MS1-level scans of the run
- Chromatogram plot via the plot generic for visual inspection

## How to apply

Obtain the raw file (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or ExperimentHub resource EH4547 in the Bioconductor package tartare) and ensure rawrr.exe plus the ThermoFisher.CommonCore RawFileReader assemblies are available (.NET 8.0 runtime on Windows, Mono on Linux/macOS). From R, call readChromatogram(rawfile, type = 'tic'); the R wrapper invokes rawrr.exe via base::system2, and the C# side iterates over all MS1-level scans, with the vendor API validating and applying scan filters so no a priori knowledge of matching scans is needed in R. Results are written to a temporary file on disk, read back, and parsed into a rawrrChromatogram S3 object, which can then be inspected with the basic print/plot generics. Optionally run rawrr's validation functions on the returned object to confirm construction. For related QC views, compute extracted ion chromatograms with readChromatogram(mass = ..., tol = 10, type = 'xic', filter = 'ms') restricted to MS1-level scans.

## Related tools

- **rawrr** (R package whose readChromatogram(type = 'tic') function drives rawrr.exe to compute and return the TIC as a rawrrChromatogram object) — https://github.com/fgcz/rawrr
- **RawFileReader** (Thermo Fisher .NET (C#) assemblies that rawrr wraps; the vendor API performs scan filtering and provides chromatographic data from the RAW file) — https://github.com/thermofisherlsms/RawFileReader
- **rawR** (Manuscript predecessor providing OS-independent access to all spectral data logged in Thermo Fisher Scientific raw files) — https://github.com/fgcz/rawR

## Examples

```
C <- rawrr::readChromatogram(rawfile, type = "tic"); plot(C)
```

## Evaluation signals

- The parsed rawrrChromatogram passes the package's validation functions
- The TIC spans the MS1-level scans of the run (for the example file, the 35 min autoQC run produced 2.1881 × 10^4 scans in total)
- TIC plot shows a chromatographic profile consistent with the iRT peptide mix in tryptic BSA digest acquired with a 20 min C18 gradient at 300 nl/min
- Chromatogram data were extracted by rawrr.exe (via system2), staged in a temporary file, and parsed into memory without prior mzML conversion
- Run header retrieved with readFileHeader() reports the expected instrument metadata for the raw file before chromatogram extraction

## Limitations

- No parallel computation support yet; large collections of raw files are processed serially
- No exchangeable backend for data access in the initial release (planned future work toward the Spectra/MsBackendRawFileReader concept)
- On Linux/macOS the pipeline depends on the Mono runtime and a correctly published rawrr.exe ('dotnet publish rawrr-dotnet.csproj --os <os> -a x64'); missing assemblies or runtime will break chromatogram extraction
- rawR was not yet submitted to Bioconductor at publication time (planned for the BioC project)

## Evidence

- [results] To get a first impression of the data we calculate a total ion chromatogram (TIC) using the readChromatogram() function.: "To get a first impression of the data we calculate a total ion chromatogram (TIC) using the readChromatogram() function."
- [results] TIC calculated from all MS1-level scans: "Total ion chromatogram (TIC) calculated from all MS1-level scans contained in 20181113_010_autoQC01.raw."
- [abstract] Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files.: "Our novel package rawR now provides complete, OS independent access to all spectral data logged in Thermo Fisher Scientific raw files."
- [methods] The file is part of the MassIVE dataset [MSV000086542]: "The file is part of the MassIVE dataset [MSV000086542]"
- [readme] rawrr wraps the functionality of the RawFileReader .NET: "rawrr wraps the functionality of the [RawFileReader](https://github.com/thermofisherlsms/RawFileReader) [.NET"
