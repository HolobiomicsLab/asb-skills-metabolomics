---
name: molecular-structure-database-search
description: Use when you have tandem mass spectra (e.g., positive-mode [M+H]+) whose
  true structures are absent from spectral libraries and you must search them against
  a large molecular structure database (LMSD, PubChem, DNP).
when_to_use_negative:
- The spectrum has an exact reference in a spectral library — spectral library search
  is the most reliable annotation method and should be preferred
- You need de novo structure prediction for unknown compounds absent from any known
  database, due to low signal-to-noise spectra and structural complexity
- Molecules are very small (<400 Da) or very heavy (>1000 Da), where the model performs
  poorly or is computationally insufficient
- You need statistical significance (p-values) for non-peptidic molecules — the MCMC
  p-value approach exists only for peptidic molecules
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: 'Primary database-search engine: scores each spectrum against all structures
    in the database using a learned probabilistic fragmentation model and ranks candidates
    by matching score'
  repo: github.com/mohimanilab/molDiscovery
- name: MetFrag
  role: 'Comparison baseline: rule-based in silico scorer using weighted peak count
    and bond dissociation energy'
- name: CFM-ID
  role: 'Comparison baseline: stochastic Markov process prediction of fragmentation
    spectra'
- name: MSFinder
  role: 'Comparison baseline: hydrogen rearrangement rules for fragmentation and scoring'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:adcf628fc0f5c84aa45a9ec684bd9a6a20fee355b522a484499b834c3c7887aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MSFinder
  - MetFrag
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# molecular-structure-database-search

## Summary

Score experimental tandem MS/MS spectra against molecular structure databases (e.g., LMSD, PubChem) using molDiscovery's probabilistic fragmentation model, then rank candidates by matching score to identify small molecules without spectral library coverage.

## When to use

Use when you have tandem mass spectra (e.g., positive-mode [M+H]+) whose true structures are absent from spectral libraries and you must search them against a large molecular structure database (LMSD, PubChem, DNP).

## When NOT to use

- The spectrum has an exact reference in a spectral library — spectral library search is the most reliable annotation method and should be preferred
- You need de novo structure prediction for unknown compounds absent from any known database, due to low signal-to-noise spectra and structural complexity
- Molecules are very small (<400 Da) or very heavy (>1000 Da), where the model performs poorly or is computationally insufficient
- You need statistical significance (p-values) for non-peptidic molecules — the MCMC p-value approach exists only for peptidic molecules

## Inputs

- Tandem MS/MS spectra (e.g., 15,917 positive-mode [M+H]+ PNNL lipid library spectra)
- Molecular structure database of candidate compounds (e.g., 39,126-compound LMSD / Lipid Maps Structure Database)
- Pretrained molDiscovery fragmentation model (trained on combined CID/HCD positive-mode GNPS spectral library reference spectra)

## Outputs

- Ranked list of candidate structures with molDiscovery matching scores for each input spectrum
- Top-1/top-3/top-5/top-10 identification accuracies (e.g., lipid_benchmark_accuracies.csv with 16.9%, 37.4%, 47.6%, 63.0%)

## How to apply

Load the tandem MS/MS spectra (e.g., 15,917 positive-mode [M+H]+ spectra from the PNNL lipid library) and the candidate structure database (e.g., the 39,126-compound LMSD). Run molDiscovery in pretrained-model database-search mode so each spectrum is scored against all candidate structures via the learned probabilistic fragmentation-matching model rather than rule-based heuristics. Rank candidate molecules per spectrum by the molDiscovery matching score and count a hit as correct when the top-ranked structure matches the known true molecule. Report top-1/3/5/10 accuracies; the paper's lipid benchmark yields 16.9%, 37.4%, 47.6%, and 63.0% respectively against LMSD. Rationale: spectral library search only covers tens of thousands of molecules, so in silico structure-database search extends identification to compounds lacking reference spectra.

## Related tools

- **molDiscovery** (Primary database-search engine: scores each spectrum against all structures in the database using a learned probabilistic fragmentation model and ranks candidates by matching score) — github.com/mohimanilab/molDiscovery
- **MetFrag** (Comparison baseline: rule-based in silico scorer using weighted peak count and bond dissociation energy)
- **CFM-ID** (Comparison baseline: stochastic Markov process prediction of fragmentation spectra)
- **MSFinder** (Comparison baseline: hydrogen rearrangement rules for fragmentation and scoring)

## Evaluation signals

- Reproduce the PNNL-lipid-vs-LMSD benchmark: top-1 = 16.9%, top-3 = 37.4%, top-5 = 47.6%, top-10 = 63.0%
- Every input spectrum receives a full ranked candidate list from the database, with scores monotonically decreasing by rank
- Known-true-molecule hits are counted only when the ranked structure matches the known true molecule; hit counts at k must be monotone non-decreasing (top-1 ≤ top-3 ≤ top-5 ≤ top-10)
- Output CSV (e.g., lipid_benchmark_accuracies.csv) contains exactly the four accuracy values in [0,1] summing consistently with hit counts / 15,917 spectra
- On the 8-million-spectrum GNPS search scale, expect ~3185 unique molecules identified at 0% FDR, a six-fold increase over prior methods

## Limitations

- Accuracy is lower for small molecules (<600 Da) because fewer fragments are generated for molecules in this range
- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) that are not fully modeled; the default model is trained on combined CID/HCD positive-mode spectra
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and is challenging to extend

## Evidence

- [abstract] Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] one can search mass spectra collected from the sample against millions of molecular structures in small molecule databases: "one can search mass spectra collected from the sample against millions of molecular structures in small molecule databases"
- [discussion] by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [discussion] This is partially because fewer fragments are generated for molecules in this range.: "This is partially because fewer fragments are generated for molecules in this range."
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
