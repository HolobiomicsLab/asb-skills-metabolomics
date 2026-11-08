---
name: molecular-formula-annotation-benchmarking
description: Use when comparing runtime or identification performance of molecular
  formula annotation tools (e.g. SIRIUS 3 vs SIRIUS 4) on fixed LC-MS/MS compound
  datasets with defined element alphabets (CHNOPS, BrClFI) and MS input modes.
when_to_use_negative:
- Input is already aggregated timing or speedup data — use direct comparison, not
  re-benchmarking
- Goal is structure identification accuracy (CSI:FingerID/CANOPUS) rather than formula-annotation
  runtime — use structure-identification benchmarking instead
- The benchmark requires GUI-based interactive analysis; this skill is for command-line,
  high-throughput runs only
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: SIRIUS 4
  role: Tool under benchmark; run in command-line mode to time fragmentation tree
    computation and isotope pattern analysis on benchmark datasets
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS
  role: Java-based software framework performing isotope pattern analysis and fragmentation
    tree computation that the benchmark times
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:9a4df14c8584b477c87b844c725e2908a3f5cd7a868181cb87a6d80ed718d36c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# molecular-formula-annotation-benchmarking

## Summary

Benchmark workflow for measuring molecular-formula identification speed and quality of SIRIUS-style fragmentation-tree/isotope-pattern tools by timing wall-clock runs on fixed compound-count datasets (e.g. 1,553 CHNOPS and 3,965 BrClFI compounds) and computing fold speedups against a reference version such as SIRIUS 3.0.

## When to use

Use when comparing runtime or identification performance of molecular formula annotation tools (e.g. SIRIUS 3 vs SIRIUS 4) on fixed LC-MS/MS compound datasets with defined element alphabets (CHNOPS, BrClFI) and MS input modes.

## When NOT to use

- Input is already aggregated timing or speedup data — use direct comparison, not re-benchmarking
- Goal is structure identification accuracy (CSI:FingerID/CANOPUS) rather than formula-annotation runtime — use structure-identification benchmarking instead
- The benchmark requires GUI-based interactive analysis; this skill is for command-line, high-throughput runs only

## Inputs

- 1,553-compound MS/MS-only benchmark dataset (CHNOPS element alphabet)
- 3,965-compound benchmark dataset with BrClFI elements (MS1+MS/MS)
- SIRIUS 4 command-line release from github.com/sirius-ms/sirius
- SIRIUS 3.0 baseline runtimes from Supplementary Results 2

## Outputs

- Per-dataset wall-clock runtime measurements
- Fold speedup values (e.g. 332-fold CHNOPS, 231-fold BrClFI)
- running_time_table.csv with runtime and speedup per dataset

## How to apply

Obtain the tool release (e.g. SIRIUS 4 from github.com/sirius-ms/sirius) and the deposited benchmark datasets underlying Supplementary Fig. 6b / Supplementary Results 2. Run the tool in command-line mode on the 1,553-compound dataset restricted to the CHNOPS element alphabet with MS/MS-only input, recording per-dataset wall-clock time for fragmentation tree computation. Repeat on the 3,965-compound dataset with BrClFI elements, enabling MS1 isotope-pattern analysis alongside MS/MS. Compute the fold speed increase relative to the SIRIUS 3.0 baseline timings (e.g. 332-fold on CHNOPS; 231-fold on BrClFI, where SIRIUS 4 took 25 min 4 s versus 4 d 1 h for the 3,850 lightest compounds). Assemble per-dataset runtime and speedup values into a running_time_table.csv; SIRIUS was faster by factors of 332 and 231 on these datasets, reflecting its improved fragmentation-tree and isotope-pattern computation over SIRIUS 3.0.

## Related tools

- **SIRIUS 4** (Tool under benchmark; run in command-line mode to time fragmentation tree computation and isotope pattern analysis on benchmark datasets) — https://github.com/sirius-ms/sirius
- **SIRIUS** (Java-based software framework performing isotope pattern analysis and fragmentation tree computation that the benchmark times) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input 1553-compounds-msms-channels.mgf --elements CHNOPS --profile qtof --output formula-results/ && sirius --input 3965-brclfi-ms1ms2.mgf --elements BrClFI --profile qtof --isotope-pattern --output brclfi-results/
```

## Evaluation signals

- Computed fold speedup reproduces reported values: 332-fold on the 1,553-compound CHNOPS dataset
- Computed fold speedup reproduces reported values: 231-fold on the 3,965-compound BrClFI dataset (25 min 4 s vs 4 d 1 h for the 3,850 lightest compounds)
- Per-dataset runtime values assemble cleanly into running_time_table.csv
- Wall-clock timings are measured in command-line mode for both tool versions on identical datasets
- Element alphabets match the reported setup: CHNOPS-only for the MS/MS-only dataset, BrClFI with MS1+MS/MS for the second

## Limitations

- Isotope pattern analysis is now rewritten and hopefully more stable than before — prior instability may affect benchmark reproducibility
- Support for in-source fragmentations and adducts is experimental
- The command line tool result output only reached its final structure in this release, so intermediate versions may produce different output formats
- ECFP fingerprints previously had to be computed on the users side; earlier tool versions require extra steps not present in the benchmarked release
- Reported speedups depend on the specific benchmark datasets and may not generalize to arbitrary compound sets

## Evidence

- [supplementary] SIRIUS 4 was 332-fold faster than SIRIUS 3.0 on the CHNOPS dataset; 231-fold faster on BrClFI (25 min 4 s vs 4 d 1 h).: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites.: [evidence span withheld — anchor status: external_doc]
- [discussion] isotope pattern analysis is now rewritten and hopefully more stable than before: "isotope pattern analysis is now rewritten and hopefully more stable than before"
- [discussion] Experimental support for in-source fragmentations and adducts: "Experimental support for in-source fragmentations and adducts"
