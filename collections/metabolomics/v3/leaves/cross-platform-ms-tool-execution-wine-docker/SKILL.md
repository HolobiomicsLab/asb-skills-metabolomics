---
name: cross-platform-ms-tool-execution-wine-docker
description: Use when the workflow requires running the Windows-only MZA converter
  on Linux/macOS (e.g., for converting Agilent .d, Bruker ion mobility .d, or Thermo
  .raw files to .mza) and the host OS is not Windows.
when_to_use_negative:
- When running natively on Windows 10 or later, where mza.exe can be executed directly
  without Wine or Docker overhead.
- When the input is not one of the supported vendor formats (.d, .raw, .mzML) — the
  executable will not recognize other MS data types.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: The Windows-native command-line executable being run cross-platform; converts
    proprietary vendor MS data into HDF5-based .mza files.
  repo: https://github.com/PNNL-m-q/mza
- name: Docker
  role: Containerization platform used to build the mzacontainer image bundling Wine
    for Unix-like execution.
- name: Wine
  role: Compatibility layer (tested version 6.0.3) that executes the Windows MZA binary
    on Linux; wine-mono.msi 7.3.0 may be required.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:fe9b13f1ecb4e3ccac30aeb692f1a545cac44b99dca5ee69f7fe4899fe0794f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - Docker
  - MZA
  - Wine
  - mz5
  - mzMLb
  - mzXML
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cross-platform-ms-tool-execution-wine-docker

## Summary

Executes the Windows-native MZA executable (mza.exe) on Unix-like systems via either a Docker container bundling Wine, or a locally installed Wine compatibility layer, enabling conversion of vendor raw MS files (.d, .raw) into .mza HDF5 files on non-Windows platforms.

## When to use

Use when the workflow requires running the Windows-only MZA converter on Linux/macOS (e.g., for converting Agilent .d, Bruker ion mobility .d, or Thermo .raw files to .mza) and the host OS is not Windows.

## When NOT to use

- When running natively on Windows 10 or later, where mza.exe can be executed directly without Wine or Docker overhead.
- When the input is not one of the supported vendor formats (.d, .raw, .mzML) — the executable will not recognize other MS data types.

## Inputs

- Vendor raw MS file (Agilent .d with or without ion mobility, Bruker ion mobility .d, Thermo .raw, or mzML)
- Optional file extension selector (-extension .mzML, .raw or .d) for batch conversion of all matching files in a directory
- Optional intensity threshold (e.g., -intensityThreshold 20) to filter signals included in the output

## Outputs

- MZA file (HDF5-based .mza) containing a metadata table plus Arrays_intensity and Arrays_mz groups (and Arrays_mzbin/Full_mz_array for IM data)
- Optionally CCScalDT/CCScalSLIM/CCScalTIMS datasets when calibration coefficients are detected during conversion

## How to apply

Choose the Docker route for reproducible, containerized execution: build the image with `docker build -t mzacontainer:latest -f docker/Dockerfile .` and run a conversion by mounting the raw-file directory and invoking `wine mza.exe` inside the container (e.g., `docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw`). Alternatively, install Wine directly (tested on Ubuntu 22.04.1 with Wine 6.0.3) and, if needed, install wine-mono.msi (tested version 7.3.0) via `wine64 uninstaller`, then call `wine mza -file <path> [-extension .d|.raw|.mzML] [-intensityThreshold N]`. Pass the same `-file` and optional `-extension`/`-intensityThreshold` arguments as on native Windows, since the executable is identical; the only platform-specific configuration is the Wine environment (set `WINEDEBUG=-all` in the Docker run to suppress debug output).

## Related tools

- **MZA** (The Windows-native command-line executable being run cross-platform; converts proprietary vendor MS data into HDF5-based .mza files.) — https://github.com/PNNL-m-q/mza
- **Docker** (Containerization platform used to build the mzacontainer image bundling Wine for Unix-like execution.)
- **Wine** (Compatibility layer (tested version 6.0.3) that executes the Windows MZA binary on Linux; wine-mono.msi 7.3.0 may be required.)

## Examples

```
docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw
```

## Evaluation signals

- The resulting .mza file opens with generic HDF5 libraries (h5py/rhdf5) and contains the expected groups (Arrays_intensity, Arrays_mz or Arrays_mzbin) and a metadata table.
- Scan identifiers in the metadata table match the HDF5 dataset names used to access each spectrum's arrays (e.g., Scan 630 -> Arrays_intensity/630).
- For IM data, IonMobilityBin=0 rows correspond to summed frame spectra and Arrays_mzbin indexes point into Full_mz_array.
- The same command-line arguments produce equivalent output on Windows and under Wine/Docker (no platform-specific divergence in converted spectra).
- Docker container exits cleanly (`--rm` removes it) and `WINEDEBUG=-all` was honored (no Wine debug spam).

## Limitations

- The MZA executable is Windows-bounded and requires Wine on Unix-like systems; wine-mono.msi must be manually installed via the uninstaller GUI for some environments.
- If the input file contains too many spectra, HDF5 group partitioning is needed; a single-group layout can slow reading performance.
- Docker execution requires building the mzacontainer image first; Wine-only execution is sensitive to the Wine version (tested 6.0.3 on Ubuntu 22.04.1).
- When converting Agilent .d files with ion mobility, ensure the appropriate calibration coefficients are present for CCS-related datasets to be written.

## Evidence

- [readme] Cross-platform execution via Docker: [evidence span withheld — anchor status: not_found]
- [readme] Wine-based execution details: [evidence span withheld — anchor status: not_found]
- [readme] docker run example: "docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw"
- [readme] Supported input formats: "Agilent '.d' (with or without ion mobility) Bruker ion mobility 'd' (with ion mobility) Thermo '.raw' mzML"
- [abstract] Cross-platform claim from abstract: "MZA provides easy, cross-platform and cross-programming language access to raw MS-data"
