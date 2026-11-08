---
name: small-molecule-identification-benchmarking
description: Use when tandem MS/MS spectra (e.g., from GNPS or MoNA spectral libraries)
  must be searched against molecular structure databases (DNP, MoNA DB, PubChem) and
  identification accuracy of multiple search methods must be compared via top-k accuracy.
when_to_use_negative:
- The goal is annotating spectra against reference spectral libraries only (spectral
  library search covers just a small fraction of known small molecules, so in silco
  database search is required instead)
- Molecules are outside molDiscovery's modeled scope, e.g., heavy molecules (>1000
  Da) where existing machine-learning approaches are computationally insufficient
- A p-value/significance score is required for non-peptidic molecules, since molDiscovery's
  MCMC p-value approach exists only for peptidic molecules
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Primary method under evaluation; performs learned probabilistic fragmentation-scoring
    search of MS/MS spectra against structure databases
  repo: github.com/mohimanilab/molDiscovery
- name: MAGMa+
  role: Competing baseline that scores both matched and missed peaks when matching
    candidate structures to spectra
- name: CFM-ID
  role: Competing baseline that predicts fragmentation spectra via a stochastic Markov
    process
- name: MetFrag
  role: Competing baseline computing matching scores from weighted peak count and
    bond dissociation energy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:a67c341e79395902521c5a83e04c3231da3c1cc8cdf8426f294fbbf5cb3cb206
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# small-molecule-identification-benchmarking

## Summary

A benchmarking skill for comparing in silco mass spectral database search methods (molDiscovery, Dereplicator+, MAGMa+, CFM-ID, CSI:FingerID, MetFrag) by top-k (top-1/3/5/10) identification accuracy on reference spectra searched against molecular structure databases such as GNPS-vs-DNP and MoNA-vs-MoNA.

## When to use

Use when tandem MS/MS spectra (e.g., from GNPS or MoNA spectral libraries) must be searched against molecular structure databases (DNP, MoNA DB, PubChem) and identification accuracy of multiple search methods must be compared via top-k accuracy.

## When NOT to use

- The goal is annotating spectra against reference spectral libraries only (spectral library search covers just a small fraction of known small molecules, so in silco database search is required instead)
- Molecules are outside molDiscovery's modeled scope, e.g., heavy molecules (>1000 Da) where existing machine-learning approaches are computationally insufficient
- A p-value/significance score is required for non-peptidic molecules, since molDiscovery's MCMC p-value approach exists only for peptidic molecules

## Inputs

- GNPS spectral library reference spectra (tandem MS/MS)
- MoNA reference spectra
- Dictionary of Natural Products (DNP) molecular structure database
- MoNA molecular structure database
- Method matching-score outputs (ranked candidate structure lists per query spectrum)

## Outputs

- top-k (top-1, top-3, top-5, top-10) accuracy values per method per benchmark setting
- Comparison table of accuracies (e.g., topk_accuracy_table.csv)
- Method ranking (e.g., molDiscovery at ~43.3% top-1 on GNPS/DNP and ~64.3% top-1 on MoNA/MoNA)

## How to apply

Set up two benchmark search settings: GNPS spectral library reference spectra searched against the Dictionary of Natural Products (DNP), and MoNA reference spectra searched against the MoNA structure database. For each query spectrum, generate candidate structures from the target database filtered by precursor mass. Run each search method — molDiscovery's learned probabilistic fragmentation-scoring model, plus competitors such as Dereplicator+, MAGMa+ (which scores both matched and missed peaks), CFM-ID (stochastic Markov process spectral prediction), CSI:FingerID, and MetFrag (weighted peak count and bond dissociation energy scoring) — and rank candidate structures per spectrum by each method's matching score. Compute top-1, top-3, top-5, and top-10 accuracies per method by checking whether the correct structure appears in the top-k ranked list, then assemble the results into a comparison table (e.g., topk_accuracy_table.csv).

## Related tools

- **molDiscovery** (Primary method under evaluation; performs learned probabilistic fragmentation-scoring search of MS/MS spectra against structure databases) — github.com/mohimanilab/molDiscovery
- **MAGMa+** (Competing baseline that scores both matched and missed peaks when matching candidate structures to spectra)
- **CFM-ID** (Competing baseline that predicts fragmentation spectra via a stochastic Markov process)
- **MetFrag** (Competing baseline computing matching scores from weighted peak count and bond dissociation energy)

## Evaluation signals

- Reproduced top-1 accuracy for molDiscovery is approximately 43.3% on the GNPS-vs-DNP benchmark and 64.3% on the MoNA-vs-MoNA benchmark
- molDiscovery ranks at or above comparison methods on MoNA and nearly matches CFM-ID (43.8% top-1) on GNPS
- Every method's accuracy is computed at all four cutoffs (top-1, top-3, top-5, top-10), with the correct structure checked against each top-k ranked list
- Output table (topk_accuracy_table.csv) contains one accuracy row per method per benchmark setting with no missing entries
- Candidate structures for each query are mass-filtered by precursor mass from the target database before scoring

## Limitations

- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules
- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and extension is challenging

## Evidence

- [abstract] molDiscovery correctly identify six times more unique small molecules than previous methods.: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [intro] MAGMa+ further scores for both matched and missed peaks: "MAGMa+ further scores for both matched and missed peaks"
- [intro] CFM-ID applies stochastic Markov process to predict fragmentation spectra: "CFM-ID applies stochastic Markov process to predict fragmentation spectra"
- [intro] MetFrag is based on weighted peak count and bond dissociation energy to compute matching scores: "MetFrag is based on weighted peak count and bond dissociation energy to compute matching scores"
- [discussion] by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit :"
