---
name: log-likelihood-ratio-spectrum-scoring
description: Use when tandem mass spectra must be matched to candidate molecules from
  a structure database (e.g., PubChem, DNP) with calibrated FDR, rather than heuristic
  peak-count scores like MetFrag or EPIC.
when_to_use_negative:
- The molecule class has a dedicated p-value framework available — for peptidic molecules,
  MCMC-based p-value computation exists, and the log-likelihood-ratio plus decoy-shuffle
  FDR is not the appropriate significance mechanism.
- 'Spectra fall outside the model''s training regime: the default model was trained
  on combined CID/HCD positive-mode spectra, so negative-mode or unusual collision-energy
  data may be poorly scored.'
- You need a spectral library search rather than in silico database search — direct
  spectral library search remains the most reliable annotation method when reference
  spectra exist for the molecule.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: Implements the trained probabilistic model, log-likelihood-ratio scoring of
    fragmentation-graph matches, decoy generation by shuffling fragmentation graphs,
    and target–decoy FDR estimation
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:4c6b9f9bb1565b3520f1bf8b48c8f579900870fed2763c3bc26a306aea3c7a65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# log-likelihood-ratio-spectrum-scoring

## Summary

Score candidate molecule–spectrum pairs with a probabilistic log-likelihood ratio and estimate false discovery rates via target–decoy shuffling, as implemented in molDiscovery for small molecule identification from tandem mass spectra.

## When to use

Use when tandem mass spectra must be matched to candidate molecules from a structure database (e.g., PubChem, DNP) with calibrated FDR, rather than heuristic peak-count scores like MetFrag or EPIC.

## When NOT to use

- The molecule class has a dedicated p-value framework available — for peptidic molecules, MCMC-based p-value computation exists, and the log-likelihood-ratio plus decoy-shuffle FDR is not the appropriate significance mechanism.
- Spectra fall outside the model's training regime: the default model was trained on combined CID/HCD positive-mode spectra, so negative-mode or unusual collision-energy data may be poorly scored.
- You need a spectral library search rather than in silico database search — direct spectral library search remains the most reliable annotation method when reference spectra exist for the molecule.

## Inputs

- Query tandem mass spectra (e.g., GNPS mzML/mgf tandem spectra)
- Candidate molecule structures from a structure database (PubChem, Dictionary of Natural Products) within 0.02 Da precursor mass tolerance
- Fragmentation graphs of candidate molecules with annotated bondType and logRank values
- Training set of matched molecule–spectrum pairs (e.g., GNPS spectral library reference spectra) with fragment logRank, bondType, and logRankpa annotations
- Trained probabilistic model parameters (CSV of smoothed conditional probabilities P(logRank|bondType, logRankpa))

## Outputs

- Log-likelihood-ratio scores for every candidate molecule–spectrum pair
- Target and decoy (shuffled fragmentation graph) score distributions
- False discovery rate estimates per score threshold
- molDiscovery_id_results.csv with target identifications, scores, and FDR estimates

## How to apply

First train P(logRank|bondType, logRankpa) by counting fragments over matched molecule–spectrum training pairs (e.g., GNPS spectral library reference spectra), applying null (uniform) normalization, and writing the learned parameters to a CSV file. For each query tandem MS/MS spectrum, retrieve structure-database candidates whose precursor mass falls within 0.02 Da of the observed precursor mass. Generate fragmentation graphs for each candidate, match fragments to spectral peaks, and compute the log-likelihood-ratio molecule–spectrum score under the trained model — this replaces heuristic bond-dissociation/peak-count scores used by MetFrag, MAGMa+, and EPIC. Generate decoy matches by shuffling the fragmentation graphs, score them identically, and derive the FDR from the target–decoy score distribution. This is the scoring core that enabled molDiscovery to identify 3185 unique small molecules at 0% FDR across 8 million GNPS spectra.

## Related tools

- **molDiscovery** (Implements the trained probabilistic model, log-likelihood-ratio scoring of fragmentation-graph matches, decoy generation by shuffling fragmentation graphs, and target–decoy FDR estimation) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- Reproduce the reported behavior of 3185 unique small molecules identified at 0% FDR in the 8-million-spectrum GNPS search.
- Trained model parameters in the output CSV must be normalized conditional probabilities: for each (bondType, logRankpa) condition, P(logRank|bondType, logRankpa) values sum to 1 after null (uniform) normalization.
- Retrieved candidate structures must all have precursor mass within 0.02 Da of the observed precursor mass.
- Decoy scores from shuffled fragmentation graphs must be computed under the identical trained model as target scores, enabling a valid target–decoy FDR estimate.
- molDiscovery_id_results.csv must contain identifications with scores and FDR estimates; identified molecule count should substantially exceed rule-based baselines (molDiscovery identifies six times more unique molecules than previous methods).

## Limitations

- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules, limiting which fragments can be scored.
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds, which distorts fragment bondType annotations feeding the model.
- Lower accuracy in the <600 Da mass range, partly because fewer fragments are generated for small molecules to score.
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra.
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and extension is challenging.

## Evidence

- [abstract] molDiscovery correctly identify six times more unique small molecules than previous methods.: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [intro] identification of 3185 molecules at 0% FDR: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] Probabilistic model learned from GNPS reference spectra: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [discussion] Covers only limited bond types and rules: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [discussion] Experimental factors not modeled: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
