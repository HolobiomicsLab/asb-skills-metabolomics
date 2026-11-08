---
name: spectral-library-artifact-generation
description: Use when a lipidomics workflow needs a custom in silico spectral library
  for novel or complex lipid classes (glycolipids, lipopeptides, FAHFA, sphingomyelin,
  ornithine lipids, cardiolipin) or for unique MSn methods, before running spectral
  matching.
when_to_use_negative:
- Input is already a completed peak table needing identification — that is the Spectrum
  Searcher or Peak Finder role, not library generation.
- User needs annotated fragment peaks from existing MS2/MS3+ matches — use the Lipid
  Spectral Annotator, not the Library Generator.
edam_operation: http://edamontology.org/operation_3567
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3174
tools:
- name: LipiDex 2 Library Generator
  role: The module that performs the skill — builds lipid classes from base templates
    and generates the in silico spectral library artifact with MSn tree-based fragmentation.
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Standalone executable (LipiDex2.exe) hosting the Library Generator module;
    requires .NET 7.0.
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:d77f7e5c2bb4a863652135f4267895cec872c2d448bd8be74f17c5f46e5e7259
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Library Generator
  edam_operation: http://edamontology.org/operation_3567
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-library-artifact-generation

## Summary

Builds an in silico lipid spectral library using the LipiDex 2 Library Generator by composing lipid classes from a base template (headgroup, backbone, up to four variable groups) and encoding MSn tree-based fragmentation logic so the library artifact is ready for Spectrum Searcher.

## When to use

Use when a lipidomics workflow needs a custom in silico spectral library for novel or complex lipid classes (glycolipids, lipopeptides, FAHFA, sphingomyelin, ornithine lipids, cardiolipin) or for unique MSn methods, before running spectral matching.

## When NOT to use

- Input is already a completed peak table needing identification — that is the Spectrum Searcher or Peak Finder role, not library generation.
- User needs annotated fragment peaks from existing MS2/MS3+ matches — use the Lipid Spectral Annotator, not the Library Generator.

## Inputs

- Lipid class definitions (headgroup, backbone, up to four variable groups)
- Variable group specifications (fatty acids, lipid classes, peptides or glycans)
- MSn tree-based fragmentation configuration (MS2 and MS3+ fragmentation logic)

## Outputs

- In silico lipid spectral library artifact (consumable by LipiDex 2 Spectrum Searcher)

## How to apply

Launch the LipiDex 2 Library Generator module (LipiDex2.exe, requires .NET 7.0). Compose each lipid class from a base template of headgroup, backbone, and up to four variable groups, where variable groups can be fatty acids, other lipid classes (e.g. cardiolipin as two PAs), or biopolymers such as peptides or glycans, per the Library Generator's simplified interface for designing complex lipid classes. Configure in silico fragmentation with support for MSn tree-based fragmentation so that MS2 and higher-order (MS3+) fragmentation logic is encoded in the library. Generate and save the spectral library artifact output for downstream consumption by the Spectrum Searcher module. Also use the generator to define the level of structural information (structural resolution) that a spectral match provides, as the article identifies this as a need the module addresses.

## Related tools

- **LipiDex 2 Library Generator** (The module that performs the skill — builds lipid classes from base templates and generates the in silico spectral library artifact with MSn tree-based fragmentation.) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Standalone executable (LipiDex2.exe) hosting the Library Generator module; requires .NET 7.0.) — https://github.com/coongroup/LipiDex-2

## Examples

```
run LipiDex2.exe → Library Generator module → define headgroup/backbone/variable groups → configure MSn tree-based fragmentation → save spectral library artifact
```

## Evaluation signals

- The generated library contains spectral entries for every defined lipid class, including classes composed via nested variable groups (e.g. cardiolipin as two PAs).
- Spectral entries carry MSn tree-based fragmentation patterns consistent with the configured MS2/MS3+ fragmentation logic.
- The library artifact is loadable by the LipiDex 2 Spectrum Searcher module without format conversion.
- Structural resolution annotations within the library reflect the intended level of structural information for each match.

## Limitations

- Library Generator is part of LipiDex2.exe, which requires .NET 7.0; the OS will prompt for this on first run.
- LipiDex 2 Peak Finder requires Compound Discoverer 3.3 .cdResult files as input for the peak table, so the generated library must be paired with a compatible downstream pipeline.
- LipiDex 1 (no longer supported) used a different Java-based library generation approach; legacy libraries may not be directly reusable in the .NET-based generator.

## Evidence
