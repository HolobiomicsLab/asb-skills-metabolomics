---
name: software-dependency-resolution-ubuntu
description: Use when a clean Ubuntu 18.04 environment must be turned into a metabolomics
  software workstation and many tools' dependencies need automated resolution instead
  of manual per-tool installation.
when_to_use_negative:
- Target OS is not Ubuntu (native support is Ubuntu-based; Windows only via virtualization
  per the User Guide)
- Only a single tool with a handful of dependencies is needed — manual installation
  is simpler than running the 103-tool catalogue installer
- Environment must remain untouched — the installer requires sudo and installs packages
  system-wide
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetumpX
  role: GUI installer that eases installation of 103 metabolomics tools by automatically
    resolving their individual dependencies and letting users choose which software
    works best for them
  repo: https://github.com/hasaniqbal777/MetumpX-bin
- name: Ubuntu
  role: Target operating system on which MetumpX performs dependency resolution and
    tool installation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btz765
    title: MetumpX—a metabolomics support package for untargeted mass spectrometry
  source_content_hash: sha256:68e6d55c29a31303c5ace26f50692faa446fed414aded719e086bdea0517b175
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hasaniqbal777/MetumpX-bin
  tools:
  - MetumpX
  - Ubuntu
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# software-dependency-resolution-ubuntu

## Summary

Automate installation of metabolomics software workstations on Ubuntu by letting a GUI installer (MetumpX) resolve each selected tool's individual dependencies — native packages plus R/Bioconductor dependencies — from a catalogue of up to 103 tools in the untargeted metabolomics mass spectrometry pipeline.

## When to use

Use when a clean Ubuntu 18.04 environment must be turned into a metabolomics software workstation and many tools' dependencies need automated resolution instead of manual per-tool installation.

## When NOT to use

- Target OS is not Ubuntu (native support is Ubuntu-based; Windows only via virtualization per the User Guide)
- Only a single tool with a handful of dependencies is needed — manual installation is simpler than running the 103-tool catalogue installer
- Environment must remain untouched — the installer requires sudo and installs packages system-wide

## Inputs

- Clean Ubuntu 18.04 execution environment (container or virtual machine)
- MetumpX installer script (MetumpX_setup_enUS) cloned from github.com/hasaniqbal777/MetumpX-bin
- User selection of desired tools from the 103-tool catalogue (GUI dialogue)

## Outputs

- Installed metabolomics software workstation with dependencies auto-resolved
- Per-tool installer outcome log (installed successfully vs. failed)
- Structured CSV table with columns tool_name, dependencies_resolved, install_status

## How to apply

Start from a clean Ubuntu execution environment (container or virtual machine), clone the MetumpX-bin repository with git, grant executable permissions to the installer with chmod, and run it with sudo to launch the GUI-based setup. Select the desired tools from the 103-tool catalogue covering the standard untargeted metabolomics mass spectrometry pipeline, allowing the installer to automatically resolve each tool's individual dependencies (native package managers and R/Bioconductor dependencies) via the on-screen dialogue boxes. Capture the per-tool outcome (installed successfully vs. failed) from the installer's dialogue and log output, and compile results into a structured CSV with columns tool_name, dependencies_resolved, and install_status. Validation: confirm the CSV contains one row per attempted tool and each install_status is 'success' or 'failure'.

## Related tools

- **MetumpX** (GUI installer that eases installation of 103 metabolomics tools by automatically resolving their individual dependencies and letting users choose which software works best for them) — https://github.com/hasaniqbal777/MetumpX-bin
- **Ubuntu** (Target operating system on which MetumpX performs dependency resolution and tool installation)

## Examples

```
git clone https://github.com/hasaniqbal777/MetumpX-bin; cd MetumpX-bin; chmod +x MetumpX_setup_enUS; sudo ./MetumpX_setup_enUS
```

## Evaluation signals

- Installer runs to completion on a clean Ubuntu 18.04 environment without manual dependency fixes
- Per-tool outcome captured for every tool in the selection list (installed successfully vs. failed)
- Compiled CSV has one row per attempted tool with columns tool_name, dependencies_resolved, install_status
- Each row's install_status is exactly 'success' or 'failure', consistent with installer's on-screen dialogue and log output
- Selected tool count does not exceed the catalogue size (103 tools in version 2.1; 89 in version 2.0)

## Limitations

- Ubuntu-based only; Windows support exists only through virtualization (see User Guide)
- Revision history shows tool count varies by version: 50 tools (v1.0), 89 (v2.0), 103 (v2.1) — pin the intended version
- Installer requires sudo and follows on-screen dialogue boxes, so it is interactive rather than fully headless
- Some tools may fail during installation; per-tool failures must be captured rather than assumed absent

## Evidence
