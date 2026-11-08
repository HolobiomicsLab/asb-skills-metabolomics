---
name: per-tool-installation-status-tracking
description: Use when installing the MetumpX toolset on Ubuntu 18.04 and you need
  to capture, per selected tool, whether dependency resolution and installation succeeded
  or failed from the installer's on-screen dialogue and logs.
when_to_use_negative:
- You only need to install a single known metabolomics tool manually rather than tracking
  bulk installation outcomes across the MetumpX catalogue
- Target OS is not Ubuntu-based and virtualization-based Windows support (per the
  User Guide) is not in scope for the tracked run
- The environment already has the tools installed and you only need dependency checks,
  not fresh installer-driven installs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MetumpX
  role: Installer whose GUI-driven setup automatically resolves individual dependencies
    for each selected tool and reports per-tool install outcomes that are tracked
    into the status table
  repo: https://github.com/hasaniqbal777/MetumpX-bin
- name: Ubuntu
  role: Base operating system on which MetumpX runs and where the per-tool installation
    statuses are captured
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btz765
    title: MetumpX—a metabolomics support package for untargeted mass spectrometry
  source_content_hash: sha256:c7f6f47a50479688c48cae8f6846eb524626f0371b4a3f17b91c69aaa3acab4c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hasaniqbal777/MetumpX-bin
  tools:
  - MetumpX
  - Ubuntu
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-tool-installation-status-tracking

## Summary

A workflow for recording, for each of the 103 tools in the MetumpX metabolomics installer, whether its individual dependencies were automatically resolved and whether installation succeeded or failed, yielding a structured per-tool status table.

## When to use

Use when installing the MetumpX toolset on Ubuntu 18.04 and you need to capture, per selected tool, whether dependency resolution and installation succeeded or failed from the installer's on-screen dialogue and logs.

## When NOT to use

- You only need to install a single known metabolomics tool manually rather than tracking bulk installation outcomes across the MetumpX catalogue
- Target OS is not Ubuntu-based and virtualization-based Windows support (per the User Guide) is not in scope for the tracked run
- The environment already has the tools installed and you only need dependency checks, not fresh installer-driven installs

## Inputs

- Clean Ubuntu 18.04 execution environment (container or virtual machine) as installation target
- MetumpX installer script MetumpX_setup_enUS with executable permissions
- MetumpX-bin repository cloned via git
- User-selected subset of the 103-tool metabolomics catalogue via the GUI dialogue
- Installer on-screen dialogue and log output per tool

## Outputs

- Structured CSV with columns tool_name, dependencies_resolved, install_status
- One CSV row per attempted tool, with install_status restricted to 'success' or 'failure'

## How to apply

Run the MetumpX_setup_enUS installer (chmod +x, then sudo ./MetumpX_setup_enUS) in a clean Ubuntu environment; the installer clones the MetumpX-bin repository and launches a GUI dialogue for selecting desired software from the 103-tool catalogue. As the installer automatically resolves each selected tool's individual dependencies (native package managers plus R/Bioconductor dependencies), capture the on-screen dialogue outcome for each tool. Record per tool: tool_name, dependencies_resolved (boolean), and install_status ('success' or 'failure'). Compile rows into a structured CSV covering every tool attempted. Validate that the CSV has one row per attempted tool and that each install_status value is exactly 'success' or 'failure', consistent with the installer's dialogue and log output.

## Related tools

- **MetumpX** (Installer whose GUI-driven setup automatically resolves individual dependencies for each selected tool and reports per-tool install outcomes that are tracked into the status table) — https://github.com/hasaniqbal777/MetumpX-bin
- **Ubuntu** (Base operating system on which MetumpX runs and where the per-tool installation statuses are captured)

## Examples

```
$ git clone https://github.com/hasaniqbal777/MetumpX-bin; cd MetumpX-bin; chmod +x MetumpX_setup_enUS; sudo ./MetumpX_setup_enUS
```

## Evaluation signals

- CSV contains a row for every tool attempted from the selection list
- Every install_status value is one of 'success' or 'failure'
- dependencies_resolved column reflects the installer's reported dependency resolution (native packages and R/Bioconductor) per tool
- Per-row outcomes are consistent with the installer's on-screen dialogue and log output
- Total rows match the number of tools selected in the GUI setup dialogue

## Limitations

- Tool catalogue size differs across MetumpX versions: v1.0 shipped 50 tools, v2.0 totaled 89 tools, and v2.1 added 15 for 103 total, so per-tool tracking scope depends on release
- Installer statuses depend on automatic dependency resolution, so failures may reflect unresolvable individual tool dependencies rather than installer defects
- Windows support is only via virtualization (see the User Guide), limiting direct per-tool status tracking on non-Ubuntu hosts

## Evidence
