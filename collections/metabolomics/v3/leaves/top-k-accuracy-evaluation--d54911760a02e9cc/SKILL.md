---
name: top-k-accuracy-evaluation
description: Use when spectra have known correct structures (spectral library or labeled
  benchmark) and you need to compare database-search methods by whether the true structure
  appears in the top-k ranked candidates.
when_to_use_negative:
- Spectra lack ground-truth structure annotations, so top-k correctness cannot be
  verified (e.g., unannotated GNPS repository searches)
- Evaluating discovery at a fixed FDR on unknown samples (e.g., the 3185 unique molecules
  at 0% FDR GNPS search), which is a different metric than top-k library accuracy
- De novo structure prediction workflows that do not produce a ranked candidate list
  from a structure database
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Primary search method whose ranked candidate lists are scored by top-k accuracy
  repo: github.com/mohimanilab/molDiscovery
- name: CFM-ID
  role: Baseline spectral-prediction search method (stochastic Markov process) compared
    in the top-k table
- name: MAGMa+
  role: Baseline method that scores both matched and missed peaks for top-k comparison
- name: MetFrag
  role: Baseline method using weighted peak count and bond dissociation energy scoring
    for top-k comparison
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:26928a06a653479a5cd6f523d5831fda2978e6d87c904de5f0bdf0b6a0e19ff4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MAGMa+
  - MetFrag
  - molDiscovery
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# top-k-accuracy-evaluation

## Summary

Protocol for benchmarking small-molecule spectral database search methods (e.g., molDiscovery vs. Dereplicator+, MAGMa+, CFM-ID, MetFrag) by computing top-1/3/5/10 identification accuracies on spectra with known ground-truth structures, as in the GNPS-vs-DNP, MoNA-vs-MoNA, and PNNL-lipid-vs-LMSD benchmarks.

## When to use

Use when spectra have known correct structures (spectral library or labeled benchmark) and you need to compare database-search methods by whether the true structure appears in the top-k ranked candidates.

## When NOT to use

- Spectra lack ground-truth structure annotations, so top-k correctness cannot be verified (e.g., unannotated GNPS repository searches)
- Evaluating discovery at a fixed FDR on unknown samples (e.g., the 3185 unique molecules at 0% FDR GNPS search), which is a different metric than top-k library accuracy
- De novo structure prediction workflows that do not produce a ranked candidate list from a structure database

## Inputs

- Tandem MS spectra with known ground-truth structures (e.g., GNPS spectral library reference spectra, MoNA reference spectra, 15,917 PNNL lipid library [M+H]+ spectra)
- Molecular structure database used for candidate generation (DNP, MoNA DB, LMSD with 39,126 compounds)
- Ranked candidate lists with matching scores from each search method (molDiscovery, MAGMa+, CFM-ID, MetFrag, Dereplicator+)

## Outputs

- Top-1, top-3, top-5, and top-10 identification accuracy percentages per method
- Comparison table of accuracies across methods (e.g., topk_accuracy_table.csv, lipid_benchmark_accuracies.csv)

## How to apply

For each query spectrum, run each search method (molDiscovery's probabilistic fragmentation scoring, plus baselines such as MAGMa+, CFM-ID, MetFrag, Dereplicator+) against a precursor-mass-filtered candidate list from the target structure database (e.g., DNP, MoNA DB, or the 39,126-compound LMSD for lipids). Rank candidate structures per spectrum by each method's matching score. Count a hit as correct when the correct structure appears among the top-1, top-3, top-5, or top-10 ranked candidates, and report the percentage of spectra at each k. Validate against reported values: molDiscovery reaches ~43.3% top-1 on GNPS-vs-DNP and 64.3% top-1 on MoNA-vs-MoNA; on the PNNL lipid benchmark (15,917 [M+H]+ spectra vs LMSD) top-1/3/5/10 = 16.9%/37.4%/47.6%/63.0%. Write the accuracy table (e.g., topk_accuracy_table.csv or lipid_benchmark_accuracies.csv) for comparison.

## Related tools

- **molDiscovery** (Primary search method whose ranked candidate lists are scored by top-k accuracy) — github.com/mohimanilab/molDiscovery
- **CFM-ID** (Baseline spectral-prediction search method (stochastic Markov process) compared in the top-k table)
- **MAGMa+** (Baseline method that scores both matched and missed peaks for top-k comparison)
- **MetFrag** (Baseline method using weighted peak count and bond dissociation energy scoring for top-k comparison)

## Evaluation signals

- Reproduced GNPS-vs-DNP benchmark shows molDiscovery top-1 accuracy of approximately 43.3% (nearly matching CFM-ID's 43.8%)
- Reproduced MoNA-vs-MoNA benchmark shows molDiscovery top-1 accuracy of approximately 64.3%, the best performer
- PNNL lipid vs LMSD benchmark reproduces top-1/3/5/10 accuracies of 16.9%/37.4%/47.6%/63.0% within reported values
- Accuracy is monotonically non-decreasing as k increases from 1 to 10 for every method
- Output table contains one top-k accuracy row per method for both benchmark settings and is saved as the expected CSV artifact

## Limitations

- Accuracy is lower in the low-mass range (<600 Da), partly because fewer fragments are generated for molecules in this range
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules, which can affect ranked scores

## Evidence

- [abstract] molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] Top-k accuracy checks on ranked candidates: "one can search mass spectra collected from the sample against millions of molecular structures in small molecule databases"
- [discussion] Lower accuracy for small molecules due to fewer fragments: "This is partially because fewer fragments are generated for molecules in this range."
- [discussion] Experimental factors influencing fragmentation accuracy: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [discussion] Learned probabilistic model from GNPS spectral library: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
