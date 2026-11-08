---
name: wine-windows-executable-deployment
description: Use when a vendor-locked Windows executable (Agilent .d, Thermo .raw
  converter) must run on Ubuntu or another Unix-like OS and no native port exists.
when_to_use_negative:
- You are already on Windows 10 or another Windows host — run the executable directly
  with no dependencies.
- The tool provides a native Linux binary or the vendor library supports cross-platform
  compilation — Wine would add unnecessary overhead.
- The task is pure data access on already-converted .mza files (just use h5py/rhdf5;
  no Wine needed).
edam_operation: http://edamontology.org/operation_3096
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: The Windows-only executable being deployed under Wine; converts vendor raw
    MS files to .mza (HDF5).
  repo: https://github.com/PNNL-m-q/mza
- name: Docker
  role: Containerizes the Wine environment so MZA can run reproducibly on Unix-like
    hosts.
- name: Wine
  role: Compatibility layer that executes the Windows binary; requires wine-mono 7.3.0
    for MZA's .NET/mono dependencies.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:54ff54e5783b1dca5ce4d2fa80b780b521be956cd3609e13eea89f6cc29b0760
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - Docker
  - MZA
  - Wine
  - mz5
  - mzXML
  edam_operation: http://edamontology.org/operation_3096
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# wine-windows-executable-deployment

## Summary

Run Windows-only command-line MS data tools (e.g., the MZA executable) on Unix-like systems by building a Wine-based Docker container or installing Wine plus wine-mono, so vendor raw files can be converted on Linux without a Windows host.

## When to use

Use when a vendor-locked Windows executable (Agilent .d, Thermo .raw converter) must run on Ubuntu or another Unix-like OS and no native port exists.

## When NOT to use

- You are already on Windows 10 or another Windows host — run the executable directly with no dependencies.
- The tool provides a native Linux binary or the vendor library supports cross-platform compilation — Wine would add unnecessary overhead.
- The task is pure data access on already-converted .mza files (just use h5py/rhdf5; no Wine needed).

## Inputs

- Windows-only command-line executable (mza.exe / mza)
- Proprietary raw MS files: Agilent '.d' (with or without ion mobility), Bruker ion mobility '.d', Thermo '.raw'
- docker/Dockerfile from the tool repository
- wine-mono.msi installer package (version 7.3.0)

## Outputs

- HDF5-based .mza files containing converted MS spectra
- Wine error/log suppression via WINEDEBUG=-all (cleaner container logs)
- A reusable Docker image tagged mzacontainer:latest

## How to apply

First decide between containerized and host-level deployment. For containers, build the provided Dockerfile (`docker build -t mzacontainer:latest -f docker/Dockerfile .`) and run with `-e WINEDEBUG=-all` to suppress Wine debug output, mounting the raw-data directory as a volume. For host deployment on Ubuntu 22.04, install Wine 6.0.3 via apt, then fetch wine-mono 7.3.0 from the official WineHQ site and register it with `wine64 uninstaller` (install from the GUI, select the .msi package, click ok). After setup, invoke the executable with `wine mza.exe -file <raw_file>` or `wine mza -file <path> -extension .raw` for batch conversion. The rationale is that MZA ships only as a Windows binary with Windows-bounded dependencies, so the Wine compatibility layer plus Mono runtime substitutes the Windows API surface needed to read Agilent, Bruker, and Thermo proprietary formats.

## Related tools

- **MZA** (The Windows-only executable being deployed under Wine; converts vendor raw MS files to .mza (HDF5).) — https://github.com/PNNL-m-q/mza
- **Docker** (Containerizes the Wine environment so MZA can run reproducibly on Unix-like hosts.)
- **Wine** (Compatibility layer that executes the Windows binary; requires wine-mono 7.3.0 for MZA's .NET/mono dependencies.)

## Examples

```
docker build -t mzacontainer:latest -f docker/Dockerfile . && docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw
```

## Evaluation signals

- Conversion produces a non-empty .mza file whose metadata table rows equal the expected number of spectra for the raw input.
- HDF5 structure check: metadata dataset plus Arrays_intensity, Arrays_mz (or Arrays_mzbin for IM spectra), and Full_mz_array groups exist and are readable via h5py.
- Running with `wine mza -file test_data -extension .d` converts all '.d' files in a directory, matching the Windows-native run's file count.
- Wine startup succeeds without 'mono' runtime errors after installing wine-mono 7.3.0.
- Container run with `WINEDEBUG=-all` produces clean logs with no Wine debug trace spam.

## Limitations

- Only tested on Windows 10 (native) and Ubuntu 22.04.1 with Wine 6.0.3 and wine-mono 7.3.0 — other versions/OS may not work.
- Wine-based execution is slower and adds container/image-management overhead relative to a native Windows run.
- MZA still has no version number and no changelog; converting very large files may hit HDF5 group-size limits requiring partition indices.
- The MZA executable itself is not open-source beyond the BSD-2-Clause license addendum; some vendor-format reading may rely on the binary.

## Evidence

- [readme] Docker build/run: [evidence span withheld — anchor status: not_found]
