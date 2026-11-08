---
name: mass-spectral-database-search
description: Use when tandem MS/MS spectra must be annotated by searching against
  a large molecular structure database (e.g., DNP, PubChem, MoNA) and candidate structures
  need probabilistic scoring plus FDR-estimated identifications.
when_to_use_negative:
- A reference spectral library contains the molecule — spectral library search is
  the most reliable annotation method and should be preferred over in silico database
  search
- De novo structure prediction is required for molecules absent from known databases
  — this skill only annotates spectra against known structure databases
- Peptidic p-value computation or experiment-specific fragmentation modeling (instrument
  type, dissociation technique, collision energy) is required — molDiscovery's default
  model covers only a limited number of bond types and rearrangement rules and was
  trained on combined CID/HCD positive-mode spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: 'Primary search engine: trains the probabilistic fragmentation model, generates
    fragmentation graphs, scores molecule-spectrum pairs, and estimates target-decoy
    FDR'
  repo: github.com/mohimanilab/molDiscovery
- name: MAGMa+
  role: Benchmark comparison method that scores both matched and missed peaks
- name: CFM-ID
  role: Benchmark comparison method applying a stochastic Markov process to predict
    fragmentation spectra
- name: MetFrag
  role: Benchmark comparison method based on weighted peak count and bond dissociation
    energy scoring
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:5d3f5f6915d34a147d5a269ecf7c06e10b83099785a3731065f26a69566c6549
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MAGMa+
  - MetFrag
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectral-database-search

## Summary

Probabilistic tandem mass spectral search of small molecules against molecular structure databases using molDiscovery, which learns a fragmentation model P(logRank|bondType, logRankpa) from training molecule-spectrum pairs and scores candidates by log-likelihood ratio with target-decoy FDR estimation.

## When to use

Use when tandem MS/MS spectra must be annotated by searching against a large molecular structure database (e.g., DNP, PubChem, MoNA) and candidate structures need probabilistic scoring plus FDR-estimated identifications.

## When NOT to use

- A reference spectral library contains the molecule — spectral library search is the most reliable annotation method and should be preferred over in silico database search
- De novo structure prediction is required for molecules absent from known databases — this skill only annotates spectra against known structure databases
- Peptidic p-value computation or experiment-specific fragmentation modeling (instrument type, dissociation technique, collision energy) is required — molDiscovery's default model covers only a limited number of bond types and rearrangement rules and was trained on combined CID/HCD positive-mode spectra

## Inputs

- Query tandem mass spectra (e.g., GNPS repository spectra, ~8 million spectra)
- Molecular structure database (e.g., DNP, MoNA DB, PubChem)
- Training molecule-spectrum pairs (e.g., GNPS spectral library reference spectra)
- Precursor mass tolerance parameter (0.02 Da)

## Outputs

- molDiscovery_id_results.csv with target identifications, scores, and FDR estimates
- Learned probabilistic model parameters written to a CSV file
- Top-k ranked candidate structures per query spectrum

## How to apply

First train the probabilistic model by counting observed fragment logRank values conditioned on bondType and logRankpa in matched molecule-spectrum training pairs (e.g., GNPS spectral library reference spectra), then normalize counts with null (uniform) normalization and write parameters to CSV. For each query tandem mass spectrum, retrieve candidate molecules from the structure database whose precursor mass falls within 0.02 Da of the observed precursor mass. Compute the log-likelihood-ratio score for every candidate molecule-spectrum pair from annotated fragmentation graph matches under the trained model. Generate decoy matches by shuffling the fragmentation graphs, score them identically, and estimate the false discovery rate from the target-decoy score distribution before writing identifications with scores and FDR to molDiscovery_id_results.csv.

## Related tools

- **molDiscovery** (Primary search engine: trains the probabilistic fragmentation model, generates fragmentation graphs, scores molecule-spectrum pairs, and estimates target-decoy FDR) — github.com/mohimanilab/molDiscovery
- **MAGMa+** (Benchmark comparison method that scores both matched and missed peaks)
- **CFM-ID** (Benchmark comparison method applying a stochastic Markov process to predict fragmentation spectra)
- **MetFrag** (Benchmark comparison method based on weighted peak count and bond dissociation energy scoring)

## Evaluation signals

- Reproduced top-1 accuracy ≈ 43.3% on the GNPS-vs-DNP benchmark and ≈ 64.3% on the MoNA-vs-MoNA benchmark (top-3, top-5, top-10 also computed)
- Searching the 8-million-spectrum GNPS repository yields 3185 unique small molecules identified at 0% FDR
- molDiscovery ranks at or above comparison methods (Dereplicator+, MAGMa+, CFM-ID, CSI:FingerID, MetFrag) in the top-k accuracy table
- Decoy matches produced by fragmentation graph shuffling show score distribution consistent with the estimated target-decoy FDR
- molDiscovery correctly identifies six times more unique small molecules than previous methods on the GNPS search

## Limitations

- Default model trained on combined CID/HCD positive-mode spectra; experimental factors such as instrument type, dissociation technique, ionization mode, and collision energy are not fully modeled
- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- No p-value computation for general molecules; MCMC p-value approach exists only for peptidic molecules and extension is challenging
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules

## Evidence

- [abstract] molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] Search of 8 million GNPS spectra identifying 3185 molecules at 0% FDR: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] Training on GNPS spectral library reference spectra to learn the probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [discussion] Known limitations: limited bond types and rearrangement rules, naive resonant structure treatment: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [readme] Academic test server contact from README: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
