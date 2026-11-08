---
name: package-manager-automation
description: Use when a clean Ubuntu 18.04 environment needs a metabolomics software
  stack installed and manually resolving each tool's dependencies would be error-prone
  or time-consuming.
when_to_use_negative:
- The target OS is not Ubuntu-based — MetumpX is explicitly a Ubuntu-based package
  (Windows is only supported via virtualization per the User Guide)
- Only a single tool with known, trivial dependencies is needed — direct installation
  is simpler than running a 103-tool catalogue installer
- The environment is not clean (pre-existing package versions may conflict with the
  automated dependency resolution)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: MetumpX
  role: Ubuntu-based installer package that automates dependency resolution for selected
    tools from its 103-tool metabolomics catalogue
  repo: https://github.com/hasaniqbal777/MetumpX-bin
- name: Ubuntu
  role: Target operating system platform on which MetumpX automates download and installation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btz765
    title: MetumpX—a metabolomics support package for untargeted mass spectrometry
  source_content_hash: sha256:49519f6015970f9119386b90804bb782f769de806bfc261843b983cecc1602e1
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# package-manager-automation

## Summary

Automates installation of metabolomics software workstations by resolving each tool's individual dependencies (native packages and R/Bioconductor libraries) from a curated catalogue, as implemented by MetumpX covering 103 untargeted mass spectrometry tools on Ubuntu.

## When to use

Use when a clean Ubuntu 18.04 environment needs a metabolomics software stack installed and manually resolving each tool's dependencies would be error-prone or time-consuming.

## When NOT to use

- The target OS is not Ubuntu-based — MetumpX is explicitly a Ubuntu-based package (Windows is only supported via virtualization per the User Guide)
- Only a single tool with known, trivial dependencies is needed — direct installation is simpler than running a 103-tool catalogue installer
- The environment is not clean (pre-existing package versions may conflict with the automated dependency resolution)

## Inputs

- Clean Ubuntu 18.04 execution environment (container or VM) with Git installed
- MetumpX-bin repository clone containing the MetumpX_setup_enUS installer script
- User's selection of desired tools from the 103-tool untargeted metabolomics catalogue

## Outputs

- Installed metabolomics software stack with all per-tool dependencies resolved
- Per-tool installation outcome (installed successfully vs. failed) from the installer's dialogue/log output
- Structured CSV table with columns tool_name, dependencies_resolved, install_status (success/failure)

## How to apply

Obtain a clean Ubuntu 18.04 target (container or VM) and ensure Git is available. Clone the MetumpX-bin repository, grant executable permissions to the installer (chmod +x MetumpX_setup_enUS), and run it with sudo; this launches a GUI-based setup presenting the 103-tool catalogue so users can select only the software that works best for them. The installer then automatically resolves each selected tool's individual dependencies, including native package-manager installs and R/Bioconductor dependencies, and reports per-tool success or failure through on-screen dialogue boxes. Capture these per-tool outcomes (e.g. into a CSV with tool_name, dependencies_resolved, install_status) and validate that every attempted tool has a row with install_status of 'success' or 'failure' consistent with the installer's log output.

## Related tools

- **MetumpX** (Ubuntu-based installer package that automates dependency resolution for selected tools from its 103-tool metabolomics catalogue) — https://github.com/hasaniqbal777/MetumpX-bin
- **Ubuntu** (Target operating system platform on which MetumpX automates download and installation)

## Examples

```
git clone https://github.com/hasaniqbal777/MetumpX-bin; cd MetumpX-bin; chmod +x MetumpX_setup_enUS; sudo ./MetumpX_setup_enUS
```

## Evaluation signals

- The CSV contains a row for every tool attempted from the selection list
- Each row's install_status is one of 'success' or 'failure', consistent with the installer's on-screen dialogue and log output
- The number of tools available for selection matches the catalogue size of the installed release (50 at v1.0, 89 at v2.0, 103 at v2.1)
- Each selected tool reports whether its individual dependencies were resolved, covering both native package-manager and R/Bioconductor dependencies
- Installer runs without prompting for manual dependency fixes — resolution is automatic per the paper's claim

## Limitations

- Ubuntu-based only; Windows support is via virtualization, not native installation (per the User Guide note in the README revision history)
- The tool count differs across sources: ~89 in the intro and README, 103 in the abstract and v2.1 release — the catalogue size depends on version
- Version 2.1 is the initial-release line with 'Bug fixes' entries but no detailed changelog
- Version 1.0 README entry repeats 'initial release' wording, so release history is not fully precise

## Evidence

- [abstract] abstract claim of 103 tools with automatic dependency resolution: "eases in the installation of 103 software by automatically resolving their individual dependencies and also allowing the users to choose which software works best for them"
- [readme] README installation commands: "# Clone this repository
$ git clone https://github.com/hasaniqbal777/MetumpX-bin

# Start Installation
$ chmod +x MetumpX_setup_enUS
$ sudo ./MetumpX_setup_enUS"
- [readme] GUI-based installation flow: "Follow the on-screen dialogue boxes for installation."
- [readme] version history showing catalogue growth to 103 tools: "**Version 2.1**
 - 15 Tools added (Total 103 Tools)
 - User Guide released
 - Added Support with Microsoft Windows using Virtualization (See User Guide)"
