---
name: cross-method-search-result-comparison
description: 'Use when evaluating or benchmarking tandem MS database search methods:
  you have query MS/MS spectra, a spectral library, a molecular structure database
  (DNP or MoNA), and need top-k accuracy comparisons across multiple search tools.'
when_to_use_negative:
- The goal is de novo structure prediction from spectra, not scoring candidates against
  a known structure database
- Query molecules are small (<400 Da) where machine-learning fragmentation methods
  are known to perform poorly, or heavy (>1000 Da) where they are computationally
  insufficient — interpret cross-method comparisons with this caveat
- Only one search method is available, making a cross-method accuracy comparison meaningless
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: 'Primary search method under benchmark: probabilistic fragmentation model
    ranks candidate structures; expected ~43.3% (GNPS) and ~64.3% (MoNA) top-1 accuracy'
  repo: github.com/mohimanilab/molDiscovery
- name: MAGMa+
  role: Comparison method that scores both matched and missed peaks for candidate
    ranking
- name: CFM-ID
  role: Comparison method applying stochastic Markov process to predict fragmentation
    spectra; best GNPS performer at 43.8% top-1
- name: MetFrag
  role: Comparison method based on weighted peak count and bond dissociation energy
    scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:f0d5f14d5f3752dfdcadb8e1b43d2f38c5bc60f68375a2c636b027e881b5d45d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-method-search-result-comparison

## Summary

Benchmark protocol for comparing small molecule identification methods (e.g., molDiscovery vs. Dereplicator+, MAGMa+, CFM-ID, CSI:FingerID, MetFrag) by ranking candidate structures per query spectrum and measuring top-k identification accuracy on GNPS and MoNA spectral library benchmarks.

## When to use

Use when evaluating or benchmarking tandem MS database search methods: you have query MS/MS spectra, a spectral library, a molecular structure database (DNP or MoNA), and need top-k accuracy comparisons across multiple search tools.

## When NOT to use

- The goal is de novo structure prediction from spectra, not scoring candidates against a known structure database
- Query molecules are small (<400 Da) where machine-learning fragmentation methods are known to perform poorly, or heavy (>1000 Da) where they are computationally insufficient — interpret cross-method comparisons with this caveat
- Only one search method is available, making a cross-method accuracy comparison meaningless

## Inputs

- GNPS spectral library reference spectra (tandem MS/MS spectra)
- MoNA reference spectra
- Dictionary of Natural Products (DNP) molecular structure database
- MoNA molecular structure database
- Search-tool outputs: ranked candidate structure lists per query spectrum from molDiscovery, Dereplicator+, MAGMa+, CFM-ID, CSI:FingerID, MetFrag

## Outputs

- topk_accuracy_table.csv with top-1, top-3, top-5, and top-10 accuracies for every method on both benchmark settings
- Per-spectrum ranked candidate structure lists keyed by method score

## How to apply

Set up two benchmark configurations: GNPS spectral library reference spectra searched against the Dictionary of Natural Products (DNP), and MoNA reference spectra searched against the MoNA structure database. For each query spectrum, generate candidate structures from the target database filtered by precursor mass, then run each search method (molDiscovery's probabilistic fragmentation scoring, plus Dereplicator+, MAGMa+, CFM-ID, CSI:FingerID, MetFrag). Rank candidate structures per spectrum by each method's matching score, then compute top-1, top-3, top-5, and top-10 accuracies by checking whether the correct structure appears in the top-k list. Assemble accuracies into a comparison table (e.g., topk_accuracy_table.csv); validation expects molDiscovery at ~43.3% top-1 on GNPS-vs-DNP and ~64.3% on MoNA-vs-MoNA, ranking at or above comparison methods (CFM-ID reaches 43.8% on GNPS).

## Related tools

- **molDiscovery** (Primary search method under benchmark: probabilistic fragmentation model ranks candidate structures; expected ~43.3% (GNPS) and ~64.3% (MoNA) top-1 accuracy) — github.com/mohimanilab/molDiscovery
- **MAGMa+** (Comparison method that scores both matched and missed peaks for candidate ranking)
- **CFM-ID** (Comparison method applying stochastic Markov process to predict fragmentation spectra; best GNPS performer at 43.8% top-1)
- **MetFrag** (Comparison method based on weighted peak count and bond dissociation energy scoring)

## Evaluation signals

- Reproduced topk_accuracy_table.csv shows molDiscovery top-1 accuracy ≈ 43.3% on GNPS-vs-DNP and ≈ 64.3% on MoNA-vs-MoNA
- molDiscovery ranks best on MoNA and within ~0.5 percentage points of CFM-ID (43.8%) on GNPS
- All six methods report top-1/top-3/top-5/top-10 accuracies computed identically as correct-structure-in-top-k over the same filtered candidate sets
- Candidate generation is precursor-mass-filtered identically for every method so accuracy differences reflect scoring, not candidate enumeration
- Each tool output is a ranked list per query spectrum; spot-check that the correct structure is present in the full candidate list before trusting accuracy numbers

## Limitations

- molDiscovery accuracy is lower in the low mass range (<600 Da), partially because fewer fragments are generated for small molecules in this range
- molDiscovery ignores local/global structural features (moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and resonant structures are naively treated as alternating single/double bonds
- Fragmentation is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Benchmark conclusions are specific to GNPS-vs-DNP and MoNA-vs-MoNA settings; library coverage is limited to tens of thousands of molecules

## Evidence

- [results] molDiscovery on average correctly identifies 43.3% and 64.3% of small molecules in the testing GNPS and MoNA data, respectively, as top-ranked identifications: [evidence span withheld — anchor status: not_found]
- [intro] CFM-ID applies stochastic Markov process to predict fragmentation spectra: "CFM-ID applies stochastic Markov process to predict fragmentation spectra"
- [intro] MAGMa+ further scores for both matched and missed peaks: "MAGMa+ further scores for both matched and missed peaks"
- [intro] MetFrag is based on weighted peak count and bond dissociation energy to compute matching scores: "MetFrag is based on weighted peak count and bond dissociation energy to compute matching scores"
- [discussion] This is partially because fewer fragments are generated for molecules in this range.: "This is partially because fewer fragments are generated for molecules in this range."
