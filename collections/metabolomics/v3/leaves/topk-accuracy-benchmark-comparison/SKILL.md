---
name: topk-accuracy-benchmark-comparison
description: Use when a search tool's identification quality must be quantified as
  top-k accuracy over spectra with known library annotations and compared against
  baseline tools reported in a reference figure or table.
when_to_use_negative:
- Spectra lack known library annotations, so no ground truth exists for computing
  top-k accuracy
- The goal is variant/discovery search (variable mode) rather than exact-mode benchmarking
  against a reference accuracy figure
- Input spectra target PubChem-scale searches where the reference baselines do not
  apply
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: VInSMoC
  role: Run in exact mode (MetaFrag/Metabolite fragmentation) in three scoring configurations
    — base, exact-peak-count, and probabilistic MCMC significance — to produce ranked
    per-scan hits for top-k accuracy computation
  repo: mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosted execution environment where VInSMoC tasks are created, parameterized
    (search mode, ion mode, tolerances, minimum score), submitted, and whose task
    page returns the best hit per input scan
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:ea02e93b7c06b0f3cee8c7f0905dbe0b16c612ebad8e289efb43997a2ac516fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# topk-accuracy-benchmark-comparison

## Summary

Benchmark a mass spectral search tool's top-k identification accuracy by ranking per-scan best hits and comparing accuracy against published baselines (e.g., Dereplicator+ and molDiscovery in Extended Data Fig. 4 of the VInSMoC paper).

## When to use

Use when a search tool's identification quality must be quantified as top-k accuracy over spectra with known library annotations and compared against baseline tools reported in a reference figure or table.

## When NOT to use

- Spectra lack known library annotations, so no ground truth exists for computing top-k accuracy
- The goal is variant/discovery search (variable mode) rather than exact-mode benchmarking against a reference accuracy figure
- Input spectra target PubChem-scale searches where the reference baselines do not apply

## Inputs

- GNPSLIB spectral library query spectra (MGF, mzXML, or mzML)
- COCONUT candidate chemical database
- NPAtlas candidate chemical database
- Published baseline top-k accuracy values (Dereplicator+ and molDiscovery, Extended Data Fig. 4)

## Outputs

- Ranked per-scan best hits with each molecule linked to its entry in the source chemical database
- vinsmoc_topk_accuracy.csv containing per-variant top-k accuracy for VInSMoC exact peak-count and probabilistic scoring vs. Dereplicator+ and molDiscovery

## How to apply

Load the query spectra (GNPSLIB library spectra in MGF/mzXML/mzML) and the candidate chemical databases (COCONUT and NPAtlas), then run VInSMoC in exact search mode using the MetaFrag/Metabolite fragmentation mode, ion mode matched to the input spectra, default precursor/product ion tolerances, and the default minimum score. Repeat the search to obtain the exact-peak-count scoring variant and the probabilistic scoring variant (statistical significance estimated via MCMC). For each input scan, retain only the best hit per spectrum that exceeded the minimum score reporting threshold, order matches by descending score, and compute top-k accuracy against the known library annotations. Tabulate the VInSMoC variant accuracies alongside the Dereplicator+ and molDiscovery baseline accuracies from Extended Data Fig. 4 to check whether ordering and values are reproduced.

## Related tools

- **VInSMoC** (Run in exact mode (MetaFrag/Metabolite fragmentation) in three scoring configurations — base, exact-peak-count, and probabilistic MCMC significance — to produce ranked per-scan hits for top-k accuracy computation) — mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosted execution environment where VInSMoC tasks are created, parameterized (search mode, ion mode, tolerances, minimum score), submitted, and whose task page returns the best hit per input scan) — https://run.npanalysis.org

## Evaluation signals

- Reproduced top-k accuracy values for VInSMoC exact peak-count and probabilistic scoring must match the ordering and values reported in Extended Data Fig. 4 relative to Dereplicator+ and molDiscovery (output_matches_reference)
- Every reported per-scan hit must exceed the configured minimum score reporting threshold
- Each reported molecule hit must link to its entry in the original chemical database (COCONUT or NPAtlas)
- Each input scan yields at most one retained best hit, ordered by descending score
- Ion mode and maximum absolute charge (e.g., 1 for [M+H]+/[M-H]-) must match the input spectra's polarity and adduct assumptions

## Limitations

- Task runs spend time in pending/running states depending on input size, so large GNPSLIB-scale searches may take substantial time to finish
- Malformed input files show an error status after parsing; the declared file extension must match the actual format
- Only the best hit per scan exceeding the minimum score is reported, so lower-ranked matches are not available for top-k evaluation beyond rank 1 from the server output
- Advanced peak-merging and peak-filtering settings should be left at defaults unless spectra were pre-processed before upload

## Evidence
