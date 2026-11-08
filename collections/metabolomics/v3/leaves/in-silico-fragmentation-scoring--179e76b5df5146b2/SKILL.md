---
name: in-silico-fragmentation-scoring
description: Use when tandem mass spectra must be matched against a molecular structure
  database (DNP, MoNA DB, PubChem) without reference spectra, and top-k identification
  accuracy is the target metric.
when_to_use_negative:
- The compound's spectrum is already present in a reference spectral library — spectral
  library search is the most reliable annotation method
- De novo structure prediction is required for molecules absent from any known structure
  database
- Accurate modeling of experimental factors (instrument type, ionization mode, collision
  energy) is essential — the default model is trained on combined CID/HCD positive-mode
  spectra only
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molDiscovery
  role: Primary probabilistic fragmentation-scoring search engine for ranking candidate
    structures against query spectra
  repo: github.com/mohimanilab/molDiscovery
- name: MAGMa+
  role: Comparison method that scores both matched and missed peaks
- name: CFM-ID
  role: Baseline method applying stochastic Markov process to predict fragmentation
    spectra
- name: MetFrag
  role: Baseline method using weighted peak count and bond dissociation energy for
    matching scores
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:0c942acf41c941f2b5c597f8c8792ae9b78771ff2eaccaec784ecbc374cc0057
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MAGMa+
  - MetFrag
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# in-silico-fragmentation-scoring

## Summary

Score candidate molecular structures against tandem MS spectra by in-silico fragment generation plus a learned probabilistic matching model, as implemented in molDiscovery for small molecule database search.

## When to use

Use when tandem mass spectra must be matched against a molecular structure database (DNP, MoNA DB, PubChem) without reference spectra, and top-k identification accuracy is the target metric.

## When NOT to use

- The compound's spectrum is already present in a reference spectral library — spectral library search is the most reliable annotation method
- De novo structure prediction is required for molecules absent from any known structure database
- Accurate modeling of experimental factors (instrument type, ionization mode, collision energy) is essential — the default model is trained on combined CID/HCD positive-mode spectra only

## Inputs

- Query tandem MS spectra (e.g., 8 million GNPS tandem mass spectra or 3,964 negative-mode GNPS spectral library spectra)
- Molecular structure database (Dictionary of Natural Products (DNP), MoNA DB, or PubChem)
- GNPS spectral library reference spectra for training the probabilistic fragmentation model

## Outputs

- Ranked list of candidate structures per query spectrum with matching scores
- Top-1/top-3/top-5/top-10 accuracy comparison table (e.g., topk_accuracy_table.csv)
- Unique molecule identifications at 0% false discovery rate (e.g., 3,185 unique small molecules from GNPS)

## How to apply

For each query spectrum, generate candidate structures from the target database filtered by precursor mass, then run molDiscovery's probabilistic fragmentation-scoring search, which combines an efficient fragment-generation algorithm with a model learned from GNPS spectral library reference spectra. Rank candidates by matching score and evaluate top-1/3/5/10 accuracy against known structures. Benchmark against Dereplicator+, MAGMa+ (which scores both matched and missed peaks), CFM-ID, CSI:FingerID, and MetFrag. Expect ~43.3% top-1 accuracy on GNPS-vs-DNP and ~64.3% on MoNA-vs-MoNA; without retraining, negative-mode GNPS spectra (3,964 queries) yield ~36% top-1 accuracy against the non-redundant DNP.

## Related tools

- **molDiscovery** (Primary probabilistic fragmentation-scoring search engine for ranking candidate structures against query spectra) — github.com/mohimanilab/molDiscovery
- **MAGMa+** (Comparison method that scores both matched and missed peaks)
- **CFM-ID** (Baseline method applying stochastic Markov process to predict fragmentation spectra)
- **MetFrag** (Baseline method using weighted peak count and bond dissociation energy for matching scores)

## Evaluation signals

- Reproduced top-1 accuracy reaches ~43.3% on the GNPS-vs-DNP benchmark and ~64.3% on MoNA-vs-MoNA, with molDiscovery best or near-best versus comparison methods
- Negative-mode evaluation without retraining gives ~36% top-1 accuracy on 3,964 GNPS negative spectra vs non-redundant DNP
- Correct structure appears within top-k ranked candidates for each query spectrum at the expected rates
- GNPS-scale search identifies ~3,185 unique small molecules at 0% FDR (sixfold increase over previous methods)
- Accuracy drops in the low mass range (<600 Da), consistent with fewer generated fragments for small molecules

## Limitations

- Lower accuracy for molecules <600 Da because fewer fragments are generated in this range
- Default model is trained on combined CID/HCD positive-mode spectra; fragmentation influenced by instrument type, ionization mode, and collision energy is not fully modeled
- Only a limited number of bond types and rearrangement rules are covered; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules

## Evidence

- [abstract] molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [abstract] Probabilistic model for matching small molecules with mass spectra: "learning a probabilistic model to match small molecules with their mass spectra"
- [intro] MAGMa+ scores both matched and missed peaks: "MAGMa+ further scores for both matched and missed peaks"
- [intro] CFM-ID applies stochastic Markov process to predict fragmentation spectra: "CFM-ID applies stochastic Markov process to predict fragmentation spectra"
- [discussion] Lower accuracy for molecules under 600 Da due to fewer generated fragments: "This is partially because fewer fragments are generated for molecules in this range."
- [discussion] Default model trained on combined CID/HCD positive-mode spectra: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [discussion] Training on GNPS spectral library reference spectra: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [readme] Academic users can test molDiscovery via the web server: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
