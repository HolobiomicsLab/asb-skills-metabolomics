---
name: in-silico-fragmentation-pattern-generation
description: Use when building an in-silico spectral library for a novel or complex
  lipid class (e.
when_to_use_negative:
- The library already exists for the lipid class and fragmentation method of interest
  — instead run Spectrum Searcher against an existing in-silico or experimental library
  such as LipidBlast.
- Peak tables or chromatographic features are already available and the goal is lipid
  identification/annotation rather than library construction — use Spectrum Searcher
  or Peak Finder instead.
- The lipid class is a simple, well-characterized species with no novel variable groups
  or non-standard MSn fragmentation scheme, where LipiDex 1 templates or public libraries
  suffice.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: LipiDex 2 Library Generator
  role: Primary module used to construct lipid classes and generate in-silico spectral
    libraries with MSn tree-based fragmentation
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Executable (LipiDex2.exe, .NET 7.0) hosting the Library Generator; also passes
    library artifacts to Spectrum Searcher
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 1
  role: Legacy predecessor whose Java-based library templates inspired the updated
    LipiDex 2 library design; no longer supported
  repo: https://github.com/coongroup/LipiDex
- name: Spectrum Searcher
  role: Downstream consumer of the generated library for MS2 and MS3+ spectral matching
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:2729e1968e1dae89098dcd33d0734a15f4c8027772449c6dedcd58ed216e726a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 1
  - LipiDex 2
  - LipiDex 2 Library Generator
  - Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# in-silico-fragmentation-pattern-generation

## Summary

Generates custom in-silico lipid spectral libraries by encoding MSn tree-based fragmentation patterns (MS2 and higher MS3+) for user-defined lipid classes in the LipiDex 2 Library Generator, so downstream Spectrum Searcher matching can identify lipids at the appropriate structural resolution.

## When to use

Use when building an in-silico spectral library for a novel or complex lipid class (e.g. glycolipids, lipopeptides, sphingomyelin, FAHFA, ornithine lipids, cardiolipin) whose fragmentation patterns are not covered by existing databases like LipidBlast or Lipid Maps, or when a targeted MSn method requires library-encoded MS3+ fragmentation logic.

## When NOT to use

- The library already exists for the lipid class and fragmentation method of interest — instead run Spectrum Searcher against an existing in-silico or experimental library such as LipidBlast.
- Peak tables or chromatographic features are already available and the goal is lipid identification/annotation rather than library construction — use Spectrum Searcher or Peak Finder instead.
- The lipid class is a simple, well-characterized species with no novel variable groups or non-standard MSn fragmentation scheme, where LipiDex 1 templates or public libraries suffice.

## Inputs

- Lipid class definitions (headgroup, backbone, up to four variable groups: fatty acids, nested lipid classes, peptides, or glycans)
- Target tandem-MS method specification (MS2 and MSn tree fragmentation conditions)

## Outputs

- In-silico lipid spectral library (.csv artifact) containing spectral entries with MSn tree-based fragmentation patterns for each defined lipid class, consumable by the LipiDex 2 Spectrum Searcher module

## How to apply

In the LipiDex 2 Library Generator (LipiDex2.exe, requires .NET 7.0), construct lipid classes from a base template of headgroup, backbone, and up to four variable groups — variable groups may be fatty acids, other lipid classes (e.g. cardiolipin as two phosphatidic acids), or biopolymers such as peptides or glycans. Configure the in-silico fragmentation engine with MSn tree-based fragmentation methods so that both MS2 spectra and higher-order (MS3+) fragmentation logic for the target tandem-MS method are encoded directly in the library. The Library Generator supports the simplified interface for designing complex lipid classes introduced in LipiDex 2 and produces the spectral library artifact consumed by the Spectrum Searcher module. Encode fragmentation at the structural-resolution level you intend to report (e.g. sum composition, fatty acyl, or sn-position), since the generated library entries define what level of structural information a spectral match can claim downstream.

## Related tools

- **LipiDex 2 Library Generator** (Primary module used to construct lipid classes and generate in-silico spectral libraries with MSn tree-based fragmentation) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Executable (LipiDex2.exe, .NET 7.0) hosting the Library Generator; also passes library artifacts to Spectrum Searcher) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 1** (Legacy predecessor whose Java-based library templates inspired the updated LipiDex 2 library design; no longer supported) — https://github.com/coongroup/LipiDex
- **Spectrum Searcher** (Downstream consumer of the generated library for MS2 and MS3+ spectral matching) — https://github.com/coongroup/LipiDex-2/releases/latest

## Examples

```
# Extract release zip and run the LipiDex 2 executable that hosts the Library Generator (requires .NET 7.0)
LipiDex2.exe
```

## Evaluation signals

- Generated library contains spectral entries for every defined lipid class, including nested classes like cardiolipin built from two phosphatidic acid variable groups.
- Library entries encode MSn tree-based fragmentation patterns covering both MS2 and MS3+ transitions consistent with the configured tandem-MS method.
- Spectrum Searcher can successfully match experimental MS2/MS3+ spectra against the generated library without library-load errors.
- Structural resolution reported for spectral matches (e.g. sum composition vs. fatty acyl vs. sn-position) corresponds to the fragmentation logic encoded in the library.
- LipiDex2.exe launches without requiring installation beyond .NET 7.0, confirming correct deployment of the Library Generator.

## Limitations

- Library Generator output is limited to lipid classes representable as headgroup + backbone + up to four variable groups; more exotic topologies may need custom template work.
- The LipiDex 2 Wiki with detailed Library Generator instructions is work-in-progress (WiP), so documentation may be incomplete.
- Requires .NET 7.0; the operating system must prompt-install it when LipiDex2.exe is first run.
- The generated library is only useful in combination with the Spectrum Searcher module; it cannot be consumed by LipiDex 1 or external search tools without adaptation.

## Evidence

- [other] Source card: Library Generator builds lipid classes from a base template of headgroup, backbone, and up to four variable groups; variable groups can be fatty acids, other lipid classes (e.g. cardiolipin as two PAs), or biopolymers such as peptides or glycans.: [evidence span withheld — anchor status: not_found]
- [other] Source card workflow step: Launch LipiDex 2 Library Generator module (part of LipiDex2.exe, requiring .NET 7.0).: [evidence span withheld — anchor status: not_found]
