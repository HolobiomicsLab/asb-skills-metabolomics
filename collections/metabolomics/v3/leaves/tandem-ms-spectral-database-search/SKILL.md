---
name: tandem-ms-spectral-database-search
description: Use when unannotated tandem MS spectra (e.g. from GNPS datasets) must
  be assigned to known small molecules in a large structure database, and in silico
  library coverage is insufficient.
when_to_use_negative:
- The molecule of interest is already represented in a reference spectral library,
  where a direct spectral library search is the most reliable annotation method
- De novo structure prediction is required because the molecule is novel and not present
  in any molecular structure database
- Data are from an experimental setup (ionization mode, dissociation technique, collision
  energy) poorly matched to the default model trained on combined CID/HCD positive-mode
  spectra
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: molDiscovery
  role: 'Primary search engine: generates fragmentations and scores molecule-spectrum
    pairs with a learned probabilistic model against the structure database'
  repo: https://github.com/mohimanilab/molDiscovery
- name: Dereplicator+
  role: Baseline in silico database search method run on the same spectra/database
    to quantify the annotation gain
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:8bc731985844263a577ed347618ef0a12908abbb1b2a64e494abf064424564bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - Dereplicator+
  - molDiscovery
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# tandem-ms-spectral-database-search

## Summary

Annotate tandem mass spectra of small molecules by searching them against large molecular structure databases (e.g. the 719,958-structure AllDB derived from PubChem/DNP) using molDiscovery's learned probabilistic fragmentation model, then filter identifications at 0% FDR.

## When to use

Use when unannotated tandem MS spectra (e.g. from GNPS datasets) must be assigned to known small molecules in a large structure database, and in silico library coverage is insufficient.

## When NOT to use

- The molecule of interest is already represented in a reference spectral library, where a direct spectral library search is the most reliable annotation method
- De novo structure prediction is required because the molecule is novel and not present in any molecular structure database
- Data are from an experimental setup (ionization mode, dissociation technique, collision energy) poorly matched to the default model trained on combined CID/HCD positive-mode spectra

## Inputs

- Tandem mass spectra (MS/MS) in GNPS/MassIVE dataset format, e.g. 46 datasets containing ~8 million spectra
- Molecular structure database of small molecules (e.g. AllDB with 719,958 structures from PubChem/DNP)
- Reference spectra from the GNPS spectral library (used to train the probabilistic fragmentation model)

## Outputs

- Spectrum-to-molecule annotations scored at a specified FDR (0% FDR reported)
- Count of matched spectra per method (e.g. molDiscovery: 56,971 spectra)
- Count of unique identified small molecules per method (e.g. molDiscovery: 3,185 vs Dereplicator+: 542)
- Results table (e.g. moldiscovery_gnps_search_results.csv)

## How to apply

Collect tandem mass spectra (e.g. ~8 million spectra from 46 GNPS MassIVE datasets such as MSV000079450, MSV000084092, MSV000086427) and a molecular structure database of small-molecule structures. Run molDiscovery, which first generates candidate fragmentations with an efficient fragmentation-generation algorithm, then scores molecule-spectrum pairs with its learned probabilistic fragmentation model (trained on GNPS spectral library reference spectra, combined CID/HCD positive mode). Run a baseline in silico tool such as Dereplicator+ on the same spectra against the same database for comparison. Filter both methods' identifications at 0% false discovery rate and tabulate matched spectra and unique compounds; in the published benchmark molDiscovery annotated 56,971 spectra and 3,185 unique compounds versus 542 unique compounds for Dereplicator+.

## Related tools

- **molDiscovery** (Primary search engine: generates fragmentations and scores molecule-spectrum pairs with a learned probabilistic model against the structure database) — https://github.com/mohimanilab/molDiscovery
- **Dereplicator+** (Baseline in silico database search method run on the same spectra/database to quantify the annotation gain)

## Evaluation signals

- At 0% FDR, molDiscovery should annotate far more spectra than the baseline (reported: 56,971 vs Dereplicator+ spectra; ~8x increase)
- Unique molecule count should approximate the reported 3,185 unique small molecules at 0% FDR (a six-fold increase over baseline's 542)
- Both methods must be filtered at the same FDR threshold (0%) before comparing counts, to make the comparison valid
- The same spectrum set and database (46 GNPS datasets, ~8M spectra; AllDB with 719,958 structures) must be used for both methods

## Limitations

- Lower annotation accuracy for molecules below ~600 Da because fewer fragments are generated for small molecules in this mass range
- molDiscovery ignores local/global structural features (e.g. moieties), covers only a limited number of bond types and rearrangement rules, and naively treats benzene rings and resonant structures as alternating single/double bonds
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules
- Fragmentation accuracy is influenced by instrument type, dissociation technique, ionization mode, and collision energy, which the default model (combined CID/HCD positive mode) does not fully model

## Evidence

- [abstract] molDiscovery is a mass spectral database search method for small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] 3,185 unique molecules at 0% FDR, six-fold increase: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] Weaker accuracy for small molecules due to fewer generated fragments: "This is partially because fewer fragments are generated for molecules in this range."
- [readme] Repository access for academic testing is via a web server, with contact email: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
