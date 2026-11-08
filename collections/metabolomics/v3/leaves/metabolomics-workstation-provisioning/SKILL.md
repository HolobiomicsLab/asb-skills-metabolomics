---
name: metabolomics-workstation-provisioning
description: Use when a clean Ubuntu 18.04 environment must be equipped with multiple
  untargeted metabolomics MS tools and manual per-tool dependency resolution is impractical;
  MetumpX automates this.
when_to_use_negative:
- The target OS is not Ubuntu-based (only Microsoft Windows virtualization support
  was added in v2.1; macOS is not mentioned)
- Only a single tool is needed and its dependencies are trivially installable by hand
- The environment cannot run GUI dialogue boxes or is not allowed sudo/root package
  installation
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetumpX
  role: Installer package whose setup script clones MetumpX-bin, presents a GUI selection
    of up to 103 tools, and automatically resolves each tool's individual dependencies
  repo: https://github.com/hasaniqbal777/MetumpX-bin
- name: Ubuntu
  role: Target OS on which MetumpX runs and into which the selected metabolomics tools
    and their dependencies are installed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btz765
    title: MetumpX—a metabolomics support package for untargeted mass spectrometry
  source_content_hash: sha256:6ca730b7c139eae74e6bb448df65989e6a188b6869f2c6724e5a5eaa4dd6476d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hasaniqbal777/MetumpX-bin
  tools:
  - MetumpX
  - Ubuntu
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolomics-workstation-provisioning

## Summary

Provision an Ubuntu-based metabolomics workstation using MetumpX, which eases installation of 103 untargeted metabolomics mass spectrometry tools by automatically resolving each tool's individual dependencies (native package managers plus R/Bioconductor) and letting the user select which software to install via a GUI.

## When to use

Use when a clean Ubuntu 18.04 environment must be equipped with multiple untargeted metabolomics MS tools and manual per-tool dependency resolution is impractical; MetumpX automates this.

## When NOT to use

- The target OS is not Ubuntu-based (only Microsoft Windows virtualization support was added in v2.1; macOS is not mentioned)
- Only a single tool is needed and its dependencies are trivially installable by hand
- The environment cannot run GUI dialogue boxes or is not allowed sudo/root package installation

## Inputs

- Clean Ubuntu 18.04 execution environment (container, virtual machine, or virtualized Windows host)
- Git installed on the target (required to clone the repository)
- Selection of desired tools from the MetumpX 103-tool catalogue made via the GUI installer

## Outputs

- Installed metabolomics software tools with dependencies automatically resolved (native packages and R/Bioconductor packages)
- Per-tool installation outcome (installed successfully vs. failed) from the installer's on-screen dialogue
- A workstation ready for the standard untargeted metabolomics mass spectrometry pipeline

## How to apply

Start from a clean Ubuntu 18.04 target (container, VM, or Windows host with virtualization per the v2.1 user guide). Clone https://github.com/hasaniqbal777/MetumpX-bin, enter the repository, and grant executable permissions to the installer with chmod +x MetumpX_setup_enUS, then run it with sudo. Follow the on-screen GUI dialogue boxes to select the desired tools from the 103-tool catalogue spanning the standard untargeted metabolomics mass spectrometry pipeline (v2.0 totalled 89 tools; v1.0 shipped 50). Let the installer automatically resolve and install each selected tool's dependencies, including R/Bioconductor packages. Record the installer's per-tool on-screen outcome (success vs. failure) for every attempted tool; the skill is judged applied correctly when every selected tool yields a logged install status and dependencies were resolved without manual intervention.

## Related tools

- **MetumpX** (Installer package whose setup script clones MetumpX-bin, presents a GUI selection of up to 103 tools, and automatically resolves each tool's individual dependencies) — https://github.com/hasaniqbal777/MetumpX-bin
- **Ubuntu** (Target OS on which MetumpX runs and into which the selected metabolomics tools and their dependencies are installed)

## Examples

```
$ git clone https://github.com/hasaniqbal777/MetumpX-bin; cd MetumpX-bin; chmod +x MetumpX_setup_enUS; sudo ./MetumpX_setup_enUS
```

## Evaluation signals

- Installer completes and reports a per-tool outcome (installed successfully vs. failed) for every tool selected from the catalogue
- Every selected tool's dependencies (including R/Bioconductor packages) were resolved automatically without manual apt/CRAN/BiocManager intervention
- The number of installed tools matches the user's GUI selection and does not exceed the catalogue total of 103
- Installer ran with sudo and executable permissions (chmod +x MetumpX_setup_enUS) as specified in the README

## Limitations

- Tool count differs across versions: v1.0 shipped 50 tools, v2.0 totalled 89, v2.1 totalled 103 — cite the version actually installed
- Ubuntu-based only; Windows support exists solely via virtualization (user guide), and no macOS support is documented
- Requires sudo and a GUI for the dialogue-based installation; headless or non-root environments may fail
- No changelog beyond the README revision history; exact installed versions of individual tools are not pinned in the provided context

## Evidence

- [abstract] MetumpX eases installation of 103 software: "this paper aims to provide MetumpX, a software package that eases in the installation of 103 software"
- [abstract] Automatic dependency resolution and user choice: "eases in the installation of 103 software by automatically resolving their individual dependencies and also allowing the users to choose which software works best for them"
- [readme] Installer commands: "$ chmod +x MetumpX_setup_enUS
$ sudo ./MetumpX_setup_enUS"
- [readme] GUI-driven installation: "GUI Installation with Selection"
- [readme] v2.1 tool count: "15 Tools added (Total 103 Tools)"
- [readme] Windows virtualization support: "Added Support with Microsoft Windows using Virtualization (See User Guide)"
