---
name: glycolipid-lipopeptide-library-entry-definition
description: Use when building an in silico spectral library for lipid classes of
  any complexity — variable groups may be fatty acyls, other lipid classes (cardiolipin
  as two PAs), or biopolymers such as peptides or glycans — and MSn fragmentation
  patterns must be defined per class.
when_to_use_negative:
- The target lipid class is a simple glycerolipid already covered by a stock spectral
  library such as LipidBlast or the Lipid Maps database — use existing resources rather
  than building a novel class entry
- Input is already a peak table or .cdResult file to be matched against an existing
  library — use Spectrum Searcher/Peak Finder, not the Library Generator
- The library design requires more than four variable groups per class, which exceeds
  the Library Generator's base template structure
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Library Generator
  role: Module used to construct lipid classes from the base template and generate
    the in silico spectral library with MSn tree-based fragmentation
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Spectrum Searcher
  role: Consumes the generated spectral library to match experimental MS2 and MS3+
    spectra against the defined lipid classes
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex (v1)
  role: Legacy module for creating and managing custom in-silico lipid spectral libraries
    via intuitive fragmentation templates
  repo: https://github.com/coongroup/LipiDex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:4ffdb2e33f2dd9383fb15b66d446419da35101b8d9796b7530c11d6031a11e84
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex (v1)
  - LipiDex 2
  - LipiDex 2 Library Generator
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# glycolipid-lipopeptide-library-entry-definition

## Summary

Defines lipid-class library entries for classes beyond simple glycerolipids — e.g. glycolipids, lipopeptides, FAHFA, ornithine lipids — by building a base template of headgroup, backbone, and up to four variable groups in the LipiDex 2 Library Generator, then encoding MSn tree-based in silico fragmentation for downstream matching.

## When to use

Use when building an in silico spectral library for lipid classes of any complexity — variable groups may be fatty acyls, other lipid classes (cardiolipin as two PAs), or biopolymers such as peptides or glycans — and MSn fragmentation patterns must be defined per class.

## When NOT to use

- The target lipid class is a simple glycerolipid already covered by a stock spectral library such as LipidBlast or the Lipid Maps database — use existing resources rather than building a novel class entry
- Input is already a peak table or .cdResult file to be matched against an existing library — use Spectrum Searcher/Peak Finder, not the Library Generator
- The library design requires more than four variable groups per class, which exceeds the Library Generator's base template structure

## Inputs

- Lipid-class definitions (headgroup, backbone, up to four variable groups per class)
- Constituent fatty acyl / head group lists for enumerating species
- Fragmentation templates (MS2 and MSn tree-based methods) per class

## Outputs

- In silico spectral library with per-class MSn tree-based fragmentation patterns
- Library entries annotated with the structural resolution level each spectral match provides
- Fragmentation logic consumable by the LipiDex 2 Spectrum Searcher for MS2/MS3+ matching

## How to apply

Launch the LipiDex 2 Library Generator module and construct the lipid class from a base template of headgroup, backbone, and up to four variable groups; this same template supports glycolipids, lipopeptides, sphingomyelin, FAHFA, ornithine lipids, and cardiolipin (modeled as two phosphatidic acid variable groups). For each class, define the structural resolution level of a spectral match (e.g. sum composition versus fatty acyl-level identification) so the Library Generator can 'also define the level of structural information that a spectral match provides'. Configure MSn tree-based fragmentation methods in the module, encoding MS2 and higher-order (MS3+) fragmentation logic so the generated library contains 'the fragmentation patterns for unique methods'. Output the library artifact for consumption by the LipiDex 2 Spectrum Searcher, which matches experimental MS2 and MS3+ spectra against it. Required-fragment logic downstream can downgrade a match to sum composition if a class-defining fragment is absent in the experimental spectrum, so ensure class entries carry those required-fragment specifications.

## Related tools

- **LipiDex 2 Library Generator** (Module used to construct lipid classes from the base template and generate the in silico spectral library with MSn tree-based fragmentation) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Spectrum Searcher** (Consumes the generated spectral library to match experimental MS2 and MS3+ spectra against the defined lipid classes) — https://github.com/coongroup/LipiDex-2
- **LipiDex (v1)** (Legacy module for creating and managing custom in-silico lipid spectral libraries via intuitive fragmentation templates) — https://github.com/coongroup/LipiDex

## Examples

```
""LipiDex2.exe → Library Generator: define headgroup + backbone + ≤4 variable groups per class; enable MSn tree-based fragmentation; export library .csv for Spectrum Searcher.""
```

## Evaluation signals

- The generated library contains spectral entries for every defined lipid class, not just a subset
- Each class entry carries MSn tree-based fragmentation patterns spanning MS2 and MS3+ transitions per the module's design
- Structural resolution level of each spectral match is encoded per class (e.g. sum composition vs. fatty acyl-level)
- Downstream Spectrum Searcher runs on the library return matches with expected required-fragment logic: a match downgrades to sum composition when the class-defining fragment is absent
- The library builds successfully in LipiDex2.exe, which requires .NET 7.0

## Limitations

- The Library Generator's base template supports at most four variable groups per lipid class, bounding class complexity
- The LipiDex 2 Wiki (instructions) is still a work in progress ('WiP'), so detailed guidance may be incomplete
- MSn tree-based fragmentation is a LipiDex 2 feature; legacy LipiDex 1 libraries do not support it
- Downstream identifications can be downgraded (e.g. plasmenyl → plasmanyl) when evidence for a higher-resolution class is insufficient

## Evidence

- [readme] Simplified interface for designing complex lipid classes: [evidence span withheld — anchor status: not_found]
- [other] variable groups can be fatty acids, other lipid classes (e.g. cardiolipin as two PAs), or biopolymers such as peptides or glycans: [evidence span withheld — anchor status: not_found]
