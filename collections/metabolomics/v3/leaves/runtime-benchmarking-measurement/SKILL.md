---
name: runtime-benchmarking-measurement
description: Use when you need to quantify the wall-clock runtime of a spectral-alignment
  or graph-based computation (e.g. MMSA minimum-spanning-tree alignment) on a molecular
  family to verify it completes in near real-time.
when_to_use_negative:
- If the input is not a molecular family of MS/MS spectra but a single pairwise alignment
  task or an already-computed molecular network output file
- If the goal is to benchmark a tool other than the MMSA greedy spanning tree alignment
  (e.g. standard pairwise spectral matching)
- If spectra have not been preprocessed with the paper's peak filtering and normalization
  steps, since timing on raw spectra would not reflect the published performance
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MMSA
  role: The greedy spanning tree alignment algorithm whose execution is being timed;
    produces the consensus peak sets for the molecular family
- name: NetworkX
  role: Constructs the minimum spanning tree of spectra used in the alignment; the
    version timed (3.3) is a performance-relevant implementation detail
- name: GNPS2
  role: Source of the rhamnolipid molecular family task data to be timed, retrieved
    by task ID
- name: metabolomics resolver
  role: Alternative source for the ~80 rhamnolipid MS/MS spectra via Universal Spectrum
    Identifiers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:efe65b5c11960dac262209bcb8ac9cd84e52c925b83476583632e180282e7210
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# runtime-benchmarking-measurement

## Summary

Benchmark technique for measuring the near real-time execution speed of the MMSA greedy spanning tree alignment on a molecular family of MS/MS spectra, e.g. the ~80-spectrum rhamnolipid family, by timing the alignment call with Python's time.perf_counter().

## When to use

Use when you need to quantify the wall-clock runtime of a spectral-alignment or graph-based computation (e.g. MMSA minimum-spanning-tree alignment) on a molecular family to verify it completes in near real-time.

## When NOT to use

- If the input is not a molecular family of MS/MS spectra but a single pairwise alignment task or an already-computed molecular network output file
- If the goal is to benchmark a tool other than the MMSA greedy spanning tree alignment (e.g. standard pairwise spectral matching)
- If spectra have not been preprocessed with the paper's peak filtering and normalization steps, since timing on raw spectra would not reflect the published performance

## Inputs

- ~80 MS/MS spectra of a molecular family (e.g. rhamnolipid family from the P. aeruginosa strains LC-MS/MS dataset sourced via GNPS2 task ID or metabolomics resolver USIs)
- Preprocessed spectra: top 6 peaks within ±25 m/z, peaks within ±17 m/z of the precursor excluded, square-root intensity transformation, normalized intensities

## Outputs

- Wall-clock runtime in seconds for the greedy spanning tree alignment of the ~80-spectrum molecular family, recorded (e.g. runtime_rhamnolipid_mmsa.json) with a note on whether the computation is near real-time (order of seconds)

## How to apply

Run the full greedy spanning tree alignment pipeline — NetworkX 3.3 MST construction with edges weighted by pairwise modified cosine similarity minus 1, topological sorting, transitive alignment of spectra pairs, and non-transitive consensus peak set computation — on the target molecular family (e.g. the ~80 MS/MS spectra of the rhamnolipid family from the P. aeruginosa strains LC-MS/MS dataset). Wrap the alignment call in Python's time.perf_counter() and execute the timing measurement repeatedly to report a representative runtime. Validate by checking that the recorded runtime for the 80-spectrum rhamnolipid family is a positive number on the order of seconds, consistent with the near real-time performance claimed in the paper, and write the measured value (e.g. to runtime_rhamnolipid_mmsa.json) with a note on whether it completes on the order of seconds.

## Related tools

- **MMSA** (The greedy spanning tree alignment algorithm whose execution is being timed; produces the consensus peak sets for the molecular family)
- **NetworkX** (Constructs the minimum spanning tree of spectra used in the alignment; the version timed (3.3) is a performance-relevant implementation detail)
- **GNPS2** (Source of the rhamnolipid molecular family task data to be timed, retrieved by task ID)
- **metabolomics resolver** (Alternative source for the ~80 rhamnolipid MS/MS spectra via Universal Spectrum Identifiers)

## Examples

```
import time; t0=time.perf_counter(); mmsa_align(rl_spectra, mst_weights=1-modified_cosine); dt=time.perf_counter()-t0; print(dt)
```

## Evaluation signals

- Recorded runtime is a positive number of seconds (near real-time) for the 80-spectrum rhamnolipid molecular family
- Timing is performed with time.perf_counter() and repeated to report a representative value
- The alignment pipeline (MST via NetworkX, topological sorting, transitive/non-transitive alignment) completes without error before the timing measurement
- Output file (e.g. runtime_rhamnolipid_mmsa.json) exists and contains a single numeric runtime value consistent with an order-of-seconds magnitude

## Limitations

- Runtime depends on implementation and environment; the paper reports performance with NetworkX 3.3, so other versions or hardware may yield different timings
- The claimed near real-time performance applies to molecular families of roughly 80 spectra (the rhamnolipid family scale); much larger families may not run in seconds
- The paper notes the greedy multiple alignment does not fully capture smaller peak alignments (e.g. y2 peaks in the albicidin family), so timing measurements on other families with complex sub-peaks may require additional manual verification of alignment completeness
- The specific GitHub repository URL for the MMSA implementation is not provided in the article text, only the local directory name NetworkFamily_MultipleAlignment_Website

## Evidence

- [results] MMSA runtime claim for 80-spectrum rhamnolipid family: [evidence span withheld — anchor status: not_found]
- [methods] Timing method and output artifact: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion: [evidence span withheld — anchor status: not_found]
- [results] Alternative data source for the benchmark input: [evidence span withheld — anchor status: not_found]
