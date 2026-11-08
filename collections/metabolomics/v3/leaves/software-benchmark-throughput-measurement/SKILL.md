---
name: software-benchmark-throughput-measurement
description: 'Use when validating a released lipidomics assay-design tool''s computational
  performance: a quantitative throughput figure (pairs per second) must be reproduced
  from a batch transition-list export on defined benchmark hardware.'
when_to_use_negative:
- The goal is validating analytical correctness (fragment masses, collision energies)
  rather than computational throughput
- LipidCreator is only available inside Skyline and no stand-alone or command-line
  mode can be invoked
- The transition list is too small to produce a stable pairs-per-second estimate comparable
  to the large experiments reported in the paper
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidCreator
  role: Runs in stand-alone batch mode to generate the precursor-product transition
    list whose export timing yields the throughput measurement
  repo: https://github.com/lifs-tools/lipidcreator/releases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:5913bd58f65c879ed7d1d8cb0a9b694cb0b1cb5af12e28ba3149c6a2f758dbd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# software-benchmark-throughput-measurement

## Summary

Measures stand-alone batch-processing throughput of LipidCreator by timing the generation of large precursor-product transition lists (>60 lipid classes) and comparing the computed pairs-per-second rate against the paper's benchmark of 60,000 pairs/sec on an Intel i5-4310M notebook with 8 GB RAM.

## When to use

Use when validating a released lipidomics assay-design tool's computational performance: a quantitative throughput figure (pairs per second) must be reproduced from a batch transition-list export on defined benchmark hardware.

## When NOT to use

- The goal is validating analytical correctness (fragment masses, collision energies) rather than computational throughput
- LipidCreator is only available inside Skyline and no stand-alone or command-line mode can be invoked
- The transition list is too small to produce a stable pairs-per-second estimate comparable to the large experiments reported in the paper

## Inputs

- LipidCreator stand-alone executable (LipidCreator.exe) from a GitHub release archive
- LipidCreator lipid knowledge-base covering the >60 supported lipid classes
- A large user-composed targeted transition list of precursor-product ion pairs

## Outputs

- Measured throughput in precursor-product pairs per second
- Wall-clock start/completion timestamps and elapsed time for the batch export
- Total count of generated precursor-product transition pairs
- Comparison record pairing the measured rate with the reported 60,000 pairs/sec benchmark

## How to apply

Install LipidCreator from the GitHub release archive and launch it in stand-alone mode (double-click LipidCreator.exe on Windows, or `mono LipidCreator.exe` on Linux). Compose a large targeted lipidomics transition list in the lipid review dialog, covering the >60 supported lipid classes with precursor-product ion pairs at a scale matching the large experiments the paper reports. Record the wall-clock start time, trigger the transition-list export to run batch generation, then record completion time, total precursor-product pairs, and elapsed seconds; compute throughput as pairs divided by elapsed seconds. Accept the measurement if it falls within the ~10^4 pairs/s order of magnitude of the paper's 60,000 pairs/sec figure, and record both measured and reported values when hardware differences prevent exact agreement.

## Related tools

- **LipidCreator** (Runs in stand-alone batch mode to generate the precursor-product transition list whose export timing yields the throughput measurement) — https://github.com/lifs-tools/lipidcreator/releases

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- Measured throughput falls within the ~10^4 pairs/s order of magnitude of the paper's 60,000 pairs/sec figure
- Elapsed time and total pair count are both recorded, and pairs/elapsed-seconds arithmetic reconciles with the reported rate
- Benchmark hardware is documented (Intel i5-4310M CPU @ 2.70 GHz, 8 GB main memory) so results are reproducible
- The generated transition list covers the >60 lipid classes supported by LipidCreator's knowledge-base

## Limitations

- Exact throughput agreement depends on hardware; differences from the reference i5-4310M/8 GB notebook must be recorded alongside the reported rate rather than treated as failure
- LipidCreator does not provide a dedicated scoring algorithm for false discovery rates and intentionally does not address coeluting or isobaric lipids, so throughput is a computational metric only
- On Linux under Mono, window repainting issues may affect the lipid review dialog workflow used to trigger the export

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [methods] Source releases are available from the GitHub release page.: "Source releases are available from the [Github release page](https://github.com/lifs-tools/lipidcreator/releases)"
- [readme] README: stand-alone operation mode.: "It also supports standalone and command-line operation."
- [readme] README: blib export from lipid review dialog after adding transitions.: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions."
