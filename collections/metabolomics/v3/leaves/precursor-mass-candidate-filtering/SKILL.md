---
name: precursor-mass-candidate-filtering
description: Use when searching query tandem mass spectra against large structure
  databases (e.g., PubChem, DNP) and you must first restrict candidate molecules by
  observed precursor mass before running fragment-matching scores.
when_to_use_negative:
- The query spectra are in negative ionization mode or from instruments/techniques
  not represented in the combined CID/HCD positive-mode training data
- The structure database contains very small molecules (<400 Da) or very heavy molecules
  (>1000 Da), where molDiscovery's fragmentation approach performs poorly or is computationally
  insufficient
- You need statistical significance for non-peptidic molecules — no p-value computation
  exists for general molecules, only an MCMC approach for peptidic ones
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: Executes the precursor-mass-restricted retrieval of candidate molecules and
    downstream probabilistic fragment-matching scoring
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:20154ce246b5dbb7be3f7eef2bbe4b40211f01c9ccad6d3e24aeec87979120bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precursor-mass-candidate-filtering

## Summary

Narrows the molecular structure database for molDiscovery spectral search by retrieving only candidate molecules whose precursor mass falls within 0.02 Da of the observed precursor mass of each query tandem mass spectrum, before probabilistic fragmentation scoring.

## When to use

Use when searching query tandem mass spectra against large structure databases (e.g., PubChem, DNP) and you must first restrict candidate molecules by observed precursor mass before running fragment-matching scores.

## When NOT to use

- The query spectra are in negative ionization mode or from instruments/techniques not represented in the combined CID/HCD positive-mode training data
- The structure database contains very small molecules (<400 Da) or very heavy molecules (>1000 Da), where molDiscovery's fragmentation approach performs poorly or is computationally insufficient
- You need statistical significance for non-peptidic molecules — no p-value computation exists for general molecules, only an MCMC approach for peptidic ones

## Inputs

- Query tandem mass spectra (e.g., from the 8-million-spectrum GNPS repository) with observed precursor masses
- Molecular structure database (e.g., PubChem, Dictionary of Natural Products)
- Trained molDiscovery fragmentation model parameters (CSV of smoothed conditional probabilities P(logRank|bondType, logRankpa))

## Outputs

- Per-spectrum candidate molecule set (precursor mass within 0.02 Da of observed precursor mass)
- Scored target identifications with log-likelihood-ratio scores and FDR estimates written to molDiscovery_id_results.csv

## How to apply

For each query tandem mass spectrum, read the observed precursor m/z and retrieve candidate molecules from the structure database (e.g., PubChem or Dictionary of Natural Products) whose precursor mass falls within 0.02 Da of the observed precursor mass. This filtering step prunes millions of database structures down to a tractable candidate set before computing log-likelihood-ratio molecule-spectrum scores under the trained P(logRank|bondType, logRankpa) model, dramatically improving search efficiency. The filter should be applied after model training and before fragmentation graph matching, and it directly gates which molecule-spectrum pairs are scored and later enter the target-decoy FDR estimation. Because the default model was trained on combined CID/HCD positive-mode spectra, precursor masses should reflect the appropriate ionization state/charge assumption.

## Related tools

- **molDiscovery** (Executes the precursor-mass-restricted retrieval of candidate molecules and downstream probabilistic fragment-matching scoring) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- Every scored molecule-spectrum pair must have |candidate precursor mass − observed precursor mass| ≤ 0.02 Da
- Final identification run on 8 million GNPS spectra should reproduce 3185 unique small molecules identified at 0% FDR
- Identified molecules should be ~6× more unique small molecules than prior methods (e.g., MetFrag, MAGa+, EPIC) on the same search
- molDiscovery_id_results.csv must contain target identifications with scores and FDR estimates for every queried spectrum
- Decoy matches generated by shuffling fragmentation graphs must be scored identically and used to estimate the false discovery rate from the target-decoy score distribution

## Limitations

- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; default model trained on combined CID/HCD positive-mode spectra

## Evidence

- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: "A search of over 8 million spectra from the Global Natural Product Social molecular networking infrastructure"
- [intro] intro: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] discussion — experimental factors: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
