---
name: spectrum-molecule-score-ranking
description: Use when tandem MS spectra (MGF, mzXML, or mzML) must be scored against
  a chemical database of candidate molecules and ranked per scan to produce top-k
  identification accuracy, including benchmarking against baseline tools like Dereplicator+
  and molDiscovery.
when_to_use_negative:
- The precursor mass difference between a query spectrum and a candidate molecule
  requires variant explanation — use VInSMoC variable search mode instead, since exact
  mode only matches molecules whose masses match the precursor m/z
- Input spectra have already been preprocessed (merged/filtered peaks) — the advanced
  peak merging and peak filtering settings should be left unchanged unless preprocessing
  has not already been done on the input
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: Mass spectral search tool that scores query mass spectra against a database
    of candidate molecules, in exact or variable search mode, and reports the best
    hit per scan exceeding the minimum score
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosted web platform (https://run.npanalysis.org) used to upload MGF/mzXML/mzML
    data, create projects and VInSMoC tasks, and view per-scan ranked results
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:c711efcb4b83ad97cd5b20dfcb6bd5726ddf52f01ef9e938cb57d0e1d77dd498
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectrum-molecule-score-ranking

## Summary

Score query mass spectra against candidate molecule databases (e.g., GNPSLIB spectra searched against COCONUT or NPAtlas) using VInSMoC, then rank the best per-scan hits by descending score and compute top-k accuracy against known annotations. This skill captures the exact-mode search, peak-count and probabilistic (MCMC statistical-significance) scoring variants, and the ranking/tabulation workflow used to benchmark against Dereplicator+ and molDiscovery.

## When to use

Use when tandem MS spectra (MGF, mzXML, or mzML) must be scored against a chemical database of candidate molecules and ranked per scan to produce top-k identification accuracy, including benchmarking against baseline tools like Dereplicator+ and molDiscovery.

## When NOT to use

- The precursor mass difference between a query spectrum and a candidate molecule requires variant explanation — use VInSMoC variable search mode instead, since exact mode only matches molecules whose masses match the precursor m/z
- Input spectra have already been preprocessed (merged/filtered peaks) — the advanced peak merging and peak filtering settings should be left unchanged unless preprocessing has not already been done on the input

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format (e.g., GNPSLIB spectral library spectra; spectra from MSV000086287)
- Candidate chemical databases (e.g., COCONUT, NPAtlas, PubChem)
- VInSMoC task parameters: search mode (exact/variable), fragmentation mode (Metabolite/Peptide), ion mode, precursor and product ion tolerances, maximum absolute charge, minimum score

## Outputs

- Per-scan best hits exceeding the minimum score reporting threshold, ranked by descending score, each linking to the molecule's entry in its source chemical database
- Top-k accuracy figures per scoring variant (peak count and probabilistic) comparable to Dereplicator+ and molDiscovery baselines (e.g., Extended Data Fig. 4; vinsmoc_topk_accuracy.csv)

## How to apply

Load the GNPSLIB (or other) spectral library spectra and the candidate chemical databases (COCONUT, NPAtlas) as inputs, then run VInSMoC hosted at run.npanalysis.org. Configure the task with exact search mode, MetaFrag/Metabolite fragmentation mode, ion mode matched to the input spectra (e.g., positive), default precursor and product ion tolerances, maximum absolute charge of 1 (so only [M+H]+ or [M-H]- adducts are used), and a minimum score reporting threshold. Run VInSMoC additional times to obtain the exact-peak-count and probabilistic scoring variants; the probabilistic variant estimates statistical significance of spectrum-molecule matches via a Markov chain Monte Carlo strategy to remove false identifications. For each input scan, retain the best hit that exceeded the minimum score threshold, order matches by descending score, and compute top-k accuracy per scoring variant against known library annotations. Optional advanced preprocessing: peak merging combines peaks within 'Peak Merge Threshold' while keeping at least 'Minimum Peak Count' peaks, and peak filtering keeps the 'Peaks Per Window' most intense peaks in m/z windows of 'Peak Filtering Window'.

## Related tools

- **VInSMoC** (Mass spectral search tool that scores query mass spectra against a database of candidate molecules, in exact or variable search mode, and reports the best hit per scan exceeding the minimum score) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosted web platform (https://run.npanalysis.org) used to upload MGF/mzXML/mzML data, create projects and VInSMoC tasks, and view per-scan ranked results) — https://run.npanalysis.org

## Evaluation signals

- Reproduced top-k accuracy for VInSMoC exact peak-count and probabilistic scoring must match the ordering and values reported in Extended Data Fig. 4 relative to Dereplicator+ and molDiscovery (output_matches_reference)
- Every reported per-scan hit must exceed the configured minimum score reporting threshold
- Each ranked hit must link to the molecule's entry in its original chemical database (COCONUT, NPAtlas, or PubChem)
- Best hit per input scan is reported exactly once, ordered by descending score
- Uploaded mass spectral data must show 'successfully parsed' status before task submission; malformed files show an error status and should be checked against the declared extension

## Limitations

- In exact mode VInSMoC only matches spectra to molecules whose masses match the precursor m/z; variants explaining precursor mass differences require variable mode
- Uploaded data must be parsed (pending state) before use; malformed inputs show an error status and the declared extension must match the actual format
- Statistical-significance filtering relies on MCMC estimation and is not guaranteed to remove all false identifications (the tool 'removes some false identifications')
- Large searches are costly at scale: the article notes current tools for database search of experimental spectra against molecule libraries are not scalable, motivating VInSMoC

## Evidence

- [results] Benchmark comparison against baselines: [evidence span withheld — anchor status: not_found]
