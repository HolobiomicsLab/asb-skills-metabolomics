---
name: mass-spectrometry-transition-pair-counting
description: Use when quantifying how fast a targeted-lipidomics tool enumerates precursor-product
  ion pairs for a transition list, or when reproducing LipidCreator's 60,000 pairs/s
  benchmark.
when_to_use_negative:
- When the goal is to analyze acquired LC-MS/MS data (PRM/SRM chromatograms) rather
  than to benchmark transition-list generation throughput
- When transitions must come from empirical spectral libraries rather than LipidCreator's
  knowledge-base-driven in silico computation of MS/MS fragment masses
- When collision-energy prediction accuracy (rather than batch-generation speed) is
  the quantity under test
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidCreator
  role: The software under benchmark; its stand-alone batch-processing mode computes
    precursor-product ion pairs from lipid names via its knowledge-base, exporting
    transition lists from the lipid review dialog.
  repo: https://github.com/lifs-tools/lipidcreator/releases
- name: Skyline
  role: Downstream integration target for the generated transition lists (PRM/MRM
    target lists and fragment libraries); also the host application through which
    LipidCreator can be launched via the Tool Store.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:1e04b458897225d08f0ab1b1a667fd5cb29971fe576b48ac64e3483507ef89ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectrometry-transition-pair-counting

## Summary

Benchmarks LipidCreator's stand-alone batch-processing mode by generating precursor-product transition pairs from lipid names and measuring throughput (paper reports 60,000 pairs/sec on an Intel i5-4310M, 8 GB RAM notebook).

## When to use

Use when quantifying how fast a targeted-lipidomics tool enumerates precursor-product ion pairs for a transition list, or when reproducing LipidCreator's 60,000 pairs/s benchmark.

## When NOT to use

- When the goal is to analyze acquired LC-MS/MS data (PRM/SRM chromatograms) rather than to benchmark transition-list generation throughput
- When transitions must come from empirical spectral libraries rather than LipidCreator's knowledge-base-driven in silico computation of MS/MS fragment masses
- When collision-energy prediction accuracy (rather than batch-generation speed) is the quantity under test

## Inputs

- Large targeted lipidomics transition list composed from LipidCreator's lipid knowledge-base covering >60 supported lipid classes (precursor-product transition pairs keyed by lipid names)
- Reference benchmark hardware specification (Intel i5-4310M CPU @ 2.70 GHz, 8 GB main memory)

## Outputs

- Measured throughput value in precursor-product pairs per second
- Wall-clock start and completion timestamps, total pair count, and elapsed seconds for the export run
- Side-by-side comparison of measured rate vs. the paper's reported 60,000 pairs/s

## How to apply

Install LipidCreator from the GitHub release page and launch it in stand-alone mode (double-click LipidCreator.exe on Windows, or 'mono LipidCreator.exe' on Linux/Ubuntu). Using the LipidCreator knowledge-base, compose a large targeted transition list spanning the >60 supported lipid classes, adding precursor-product pairs until the list scale matches the large experiments the paper reports. Trigger the transition-list export from the lipid review dialog (as used for blib spectral library export), recording the wall-clock start time before export and the completion time after. Compute throughput as total precursor-product pairs divided by elapsed seconds. Accept the result if the measured rate falls in the ~10^4 pairs/s order of magnitude reported by the paper (60,000 pairs/s); otherwise record both the measured and reported rates, noting hardware differences.

## Related tools

- **LipidCreator** (The software under benchmark; its stand-alone batch-processing mode computes precursor-product ion pairs from lipid names via its knowledge-base, exporting transition lists from the lipid review dialog.) — https://github.com/lifs-tools/lipidcreator/releases
- **Skyline** (Downstream integration target for the generated transition lists (PRM/MRM target lists and fragment libraries); also the host application through which LipidCreator can be launched via the Tool Store.) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- Measured throughput falls in the ~10^4 pairs/s order of magnitude reported in the paper (60,000 precursor-product pairs per second)
- Transition list scale matches the large targeted experiments the paper describes, covering the >60 supported lipid classes
- Elapsed time is computed from recorded wall-clock start and completion timestamps bracketing the transition-list export step
- The pair count reported equals the number of precursor-product pairs written by the export, not an estimate from input lipid names
- Benchmark hardware (CPU model, clock speed, RAM size) is recorded alongside the measured rate for reproducibility

## Limitations

- The 60,000 pairs/s reference rate was measured on a specific notebook (Intel i5-4310M @ 2.70 GHz, 8 GB RAM); different hardware will yield different absolute rates, so only the order of magnitude is an acceptance criterion
- Under Linux/Mono there may be issues with repainting of certain windows and controls (scrollable areas) due to the not 100% compatible Mono implementation
- Blib spectral library export requires sqlite (>3); on Linux a precompiled native libSQLite.Interop.so ships with LipidCreator, and if it fails a custom one must be compiled
- The throughput metric counts precursor-product pair computation only; it does not reflect collision-energy prediction or spectral-library writing costs

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [readme] It also supports standalone and command-line operation.: "It also supports standalone and command-line operation."
- [methods] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [methods] Source releases are available from the Github release page: "Source releases are available from the [Github release page](https://github.com/lifs-tools/lipidcreator/releases)"
