---
name: probabilistic-fragmentation-model-training
description: Use when training or retraining a learned fragmentation model for tandem
  mass spectral database search, or when scoring candidate molecule-spectrum pairs
  and estimating target-decoy FDR for small molecule identification.
when_to_use_negative:
- The task is de novo structure prediction or pure spectral library search rather
  than in silico search against a molecular structure database
- Accurate modeling of low-mass molecules (<600 Da) is critical, since fewer fragments
  are generated for molecules in this range and accuracy is lower
- Experimental conditions (instrument type, dissociation technique, ionization mode,
  collision energy) differ substantially from the default model, which was trained
  on combined CID/HCD positive-mode spectra
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molDiscovery
  role: Trains the fragment-matching probabilistic model, scores molecule-spectrum
    pairs via log-likelihood ratio, generates shuffled decoy matches for FDR estimation,
    and outputs molDiscovery_id_results.csv
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:cb7641718d2aae66ac0e84b1d5195126b86aa12e048ac5a65131d00a412be30a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# probabilistic-fragmentation-model-training

## Summary

Train the molDiscovery probabilistic fragment-matching model, P(logRank|bondType, logRankpa), by counting observed fragment bondType/logRank pairs in matched molecule-spectrum training pairs and normalizing the count matrix, then use it for log-likelihood-ratio scoring with target-decoy FDR estimation.

## When to use

Use when training or retraining a learned fragmentation model for tandem mass spectral database search, or when scoring candidate molecule-spectrum pairs and estimating target-decoy FDR for small molecule identification.

## When NOT to use

- The task is de novo structure prediction or pure spectral library search rather than in silico search against a molecular structure database
- Accurate modeling of low-mass molecules (<600 Da) is critical, since fewer fragments are generated for molecules in this range and accuracy is lower
- Experimental conditions (instrument type, dissociation technique, ionization mode, collision energy) differ substantially from the default model, which was trained on combined CID/HCD positive-mode spectra

## Inputs

- Matched molecule-spectrum training pairs (e.g., GNPS spectral library reference spectra with known structures)
- Query tandem mass spectra (e.g., GNPS repository spectra)
- Molecular structure database (e.g., PubChem, Dictionary of Natural Products) for candidate retrieval
- Null normalization setting for count-matrix smoothing

## Outputs

- Trained probabilistic model parameters P(logRank|bondType, logRankpa) written to a CSV file
- Log-likelihood-ratio molecule-spectrum scores for all candidate pairs
- molDiscovery_id_results.csv with target identifications, scores, and FDR estimates

## How to apply

Parse matched molecule-spectrum training pairs (e.g., reference spectra from the GNPS spectral library) and build a count matrix over fragment logRank values conditioned on bondType and logRankpa using the molDiscovery training module. Train P(logRank|bondType, logRankpa) by normalizing the counts with null (uniform) normalization to obtain smoothed conditional probabilities, and write the learned parameters to a CSV file. For each query tandem mass spectrum, retrieve candidate molecules from the structure database (e.g., PubChem or DNP) whose precursor mass falls within 0.02 Da of the observed precursor mass, compute the log-likelihood-ratio score for every candidate molecule-spectrum pair from fragmentation graph matches, generate decoy matches by shuffling the fragmentation graphs, score them identically, and estimate the FDR from the target-decoy score distribution.

## Related tools

- **molDiscovery** (Trains the fragment-matching probabilistic model, scores molecule-spectrum pairs via log-likelihood ratio, generates shuffled decoy matches for FDR estimation, and outputs molDiscovery_id_results.csv) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- The learned-parameter CSV contains normalized conditional probabilities (each P(logRank|bondType, logRankpa) within [0,1]) obtained from count normalization
- Precursor-mass filtering retains only candidates within 0.02 Da of the observed precursor mass
- Target-decoy score distributions yield a valid FDR estimate; benchmark is 3185 unique small molecules identified at 0% FDR in the 8-million-spectrum GNPS search
- molDiscovery_id_results.csv is produced with identifications, scores, and FDR columns

## Limitations

- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and extension is challenging
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules

## Evidence

- [abstract] learning a probabilistic model to match small molecules with their mass spectra: "learning a probabilistic model to match small molecules with their mass spectra"
- [discussion] by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [intro] molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
