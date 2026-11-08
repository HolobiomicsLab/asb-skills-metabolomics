---
name: computational-performance-validation
description: Use when a paper reports a concrete computational throughput or runtime
  figure (e.g. 60,000 transition pairs/sec) and you must reproduce or sanity-check
  that number on available hardware before trusting the tool at scale.
when_to_use_negative:
- The question is about transition-list correctness (fragment masses, collision-energy
  predictions, blib spectral library output) rather than computation speed — validate
  content separately, e.g. by exporting a blib file after activating the collision
  energy module.
- The reported figure concerns instrument-side acquisition or analytical validation
  (PRM/SRM on Q-Exactive HF or QTRAP 6500, NIST SRM 1950 plasma analysis), not software
  runtime — timing LipidCreator does not speak to those claims.
- Only a small transition list (a handful of lipids) is needed; per-pair throughput
  is not measurable at that scale and the order-of-magnitude comparison becomes meaningless.
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: LipidCreator
  role: 'Tool under benchmark: run in stand-alone mode to batch-generate precursor-product
    ion pairs from lipid names across >60 lipid classes and time the transition-list
    export'
  repo: https://github.com/lifs-tools/lipidcreator/releases
- name: Skyline
  role: Host application with which LipidCreator is integrated; the stand-alone LipidCreator.exe
    can also be found in the Skyline installation's Tools folder
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:1b4878df9388539596b8b99f1eebe01dbbb1452c1a71962bbf7200a52e0cb44f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# computational-performance-validation

## Summary

Benchmarks the computational throughput of a tool against a paper-reported figure, here measuring how many precursor-product ion pairs per second LipidCreator's stand-alone batch mode generates versus the reported 60,000 pairs/sec on an i5-4310M notebook with 8 GB RAM.

## When to use

Use when a paper reports a concrete computational throughput or runtime figure (e.g. 60,000 transition pairs/sec) and you must reproduce or sanity-check that number on available hardware before trusting the tool at scale.

## When NOT to use

- The question is about transition-list correctness (fragment masses, collision-energy predictions, blib spectral library output) rather than computation speed — validate content separately, e.g. by exporting a blib file after activating the collision energy module.
- The reported figure concerns instrument-side acquisition or analytical validation (PRM/SRM on Q-Exactive HF or QTRAP 6500, NIST SRM 1950 plasma analysis), not software runtime — timing LipidCreator does not speak to those claims.
- Only a small transition list (a handful of lipids) is needed; per-pair throughput is not measurable at that scale and the order-of-magnitude comparison becomes meaningless.

## Inputs

- LipidCreator stand-alone build (LipidCreator.exe) from the released source archive at github.com/lifs-tools/lipidcreator/releases
- LipidCreator knowledge-base covering >60 lipid classes, used to compose a large targeted transition list of precursor-product pairs
- Benchmark hardware specification from the paper (Intel i5-4310M CPU @ 2.70 GHz, 8 GB main memory) as the reference point
- Wall-clock timestamps recorded immediately before and after the transition-list export

## Outputs

- Measured throughput in precursor-product pairs per second
- Total number of precursor-product pairs generated
- Elapsed wall-clock seconds for the batch export
- Comparison record pairing the measured rate with the paper's reported rate of 60,000 pairs/sec (e.g. in the output JSON)

## How to apply

Install the tool from its released source archive (for LipidCreator, the Github release page) and launch it in stand-alone mode — e.g. double-click LipidCreator.exe or run 'mono LipidCreator.exe' on Linux. Compose a targeted lipidomics transition list spanning the >60 supported lipid classes via the LipidCreator knowledge-base, scaling the precursor-product pair count to match the large experiments the paper describes. Trigger batch generation of the full transition list via the lipid review dialog's transition-list export, recording the wall-clock start time before export and the completion time after. Compute throughput as (total precursor-product pairs) / (elapsed seconds) and compare against the paper's reported figure (60,000 pairs/sec on an Intel i5-4310M CPU @ 2.70 GHz, 8 GB RAM). Because benchmark hardware may differ, accept the result if the measured rate falls within the reported order of magnitude (~10^4 pairs/sec), and always record both the measured and the reported rate in the output.

## Related tools

- **LipidCreator** (Tool under benchmark: run in stand-alone mode to batch-generate precursor-product ion pairs from lipid names across >60 lipid classes and time the transition-list export) — https://github.com/lifs-tools/lipidcreator/releases
- **Skyline** (Host application with which LipidCreator is integrated; the stand-alone LipidCreator.exe can also be found in the Skyline installation's Tools folder) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- Measured throughput falls within the order of magnitude of the reported 60,000 pairs/sec (i.e. in the ~10^4 pairs/sec range).
- Both the measured rate and the paper's reported 60,000 pairs/sec are explicitly recorded side by side in the output JSON.
- The total precursor-product pair count and elapsed seconds are logged, and throughput equals pairs divided by elapsed seconds.
- The benchmark ran LipidCreator in stand-alone batch mode (not inside Skyline) with a transition list large enough to reflect the paper's large targeted experiments.
- The comparison notes hardware differences from the reference i5-4310M / 8 GB RAM configuration whenever exact agreement is not achieved.

## Limitations

- The reported rate is hardware-specific (i5-4310M CPU @ 2.70 GHz, 8 GB RAM); other machines can only be validated to order-of-magnitude agreement.
- On Linux under Mono there may be issues with repainting of certain windows and controls due to the not 100% compatible Mono implementation, which could affect interactive export timing.
- LipidCreator intentionally does not provide a dedicated scoring algorithm for false discovery rates and does not address coeluting/isobaric lipids or peak area integration — throughput says nothing about these analytical limitations.
- Collision energy models are currently exclusive to lipid mediators and coverage of eukaryotic lipid classes has exceptions (Supplementary Data 1), so the transition list composition affects what is being timed.

## Evidence

- [full_text] LipidCreator's stand-alone batch-processing mode computes precursor-product ion pairs at a rate of 60,000 pairs per second on a standard notebook with an i5-4310M CPU @ 2.70 GHz and 8 GB of main memory.: [evidence span withheld — anchor status: not_found]
- [readme] It also supports standalone and command-line operation.: "It also supports standalone and command-line operation."
- [readme] Source releases are available from the [Github release page](https://github.com/lifs-tools/lipidcreator/releases): "Source releases are available from the [Github release page](https://github.com/lifs-tools/lipidcreator/releases)"
- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [methods] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
