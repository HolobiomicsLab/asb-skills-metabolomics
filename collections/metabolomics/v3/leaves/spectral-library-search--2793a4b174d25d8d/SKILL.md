---
name: spectral-library-search
description: Use when query tandem MS spectra must be matched to candidate molecular
  structures in a database (e.g., GNPS spectral library, DNP, PubChem, LMSD) to identify
  small molecules or lipids.
when_to_use_negative:
- The query molecules are already represented in a reference spectral library — direct
  spectral library search is 'the most reliable annotation method' and preferred over
  in silico database search
- The spectra require de novo structure prediction of unknown/novel molecules not
  present in any structure database
- Molecules are super small (<400 Da) or very heavy (>1000 Da), where existing machine
  learning scoring approaches perform poorly or are computationally insufficient
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: 'Primary search engine: generates fragmentations and scores molecule-spectrum
    pairs with a learned probabilistic model to identify small molecules from spectra'
  repo: github.com/mohimanilab/molDiscovery
- name: manual expert review
  role: Expert curation of identification results when validating benchmark accuracies
    (lipid LMSD benchmark)
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:c992ed2364add36d85be440cb38364cf5191972a059b7cd22c4f04ee9a020137
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - manual expert review
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spectral-library-search

## Summary

A mass spectral database-search skill for small molecule identification that scores query tandem MS spectra against candidate structures using molDiscovery's learned probabilistic fragmentation model, as applied to GNPS, DNP, LMSD, and PNNL lipid benchmarks.

## When to use

Use when query tandem MS spectra must be matched to candidate molecular structures in a database (e.g., GNPS spectral library, DNP, PubChem, LMSD) to identify small molecules or lipids.

## When NOT to use

- The query molecules are already represented in a reference spectral library — direct spectral library search is 'the most reliable annotation method' and preferred over in silico database search
- The spectra require de novo structure prediction of unknown/novel molecules not present in any structure database
- Molecules are super small (<400 Da) or very heavy (>1000 Da), where existing machine learning scoring approaches perform poorly or are computationally insufficient

## Inputs

- Tandem MS (MS/MS) spectra of query small molecules (e.g., 3,964 negative-mode or 15,917 positive-mode [M+H]+ spectra from GNPS/PNNL libraries)
- Molecular structure database of candidate compounds (e.g., unique/non-redundant DNP, LMSD 39,126 lipids, PubChem)
- Optional: trained molDiscovery probabilistic fragmentation model (default pretrained on combined CID/HCD positive-mode spectra)

## Outputs

- Ranked list of candidate structures per query spectrum scored by molDiscovery matching score
- Top-1 (and top-3/5/10) identification accuracies vs. known annotations (e.g., lipid_benchmark_accuracies.csv)
- Unique small molecule identifications at a controlled FDR (e.g., 3,185 unique molecules at 0% FDR from 8 million GNPS spectra)

## How to apply

Run molDiscovery's database search, which (i) generates mass spectrometry fragmentations for each candidate structure and (ii) scores molecule-spectrum pairs with a probabilistic model learned from GNPS spectral library reference spectra. For each query spectrum, rank all candidate structures by the molDiscovery matching score and take the top-scoring candidate as the identification. Validate performance by computing top-1 (and optionally top-3/5/10) accuracy against known spectral library annotations — e.g., ~36% on 3,964 negative-mode GNPS spectra against unique DNP using the default model, and top-1 = 16.9% / top-3 = 37.4% / top-5 = 47.6% / top-10 = 63.0% for 15,917 PNNL lipid [M+H]+ spectra against the 39,126-compound LMSD. For best accuracy, retrain the probabilistic model on reference spectra matching the experimental ionization mode, since the default model was trained on combined CID/HCD positive-mode spectra.

## Related tools

- **molDiscovery** (Primary search engine: generates fragmentations and scores molecule-spectrum pairs with a learned probabilistic model to identify small molecules from spectra) — github.com/mohimanilab/molDiscovery
- **manual expert review** (Expert curation of identification results when validating benchmark accuracies (lipid LMSD benchmark))

## Examples

```
molDiscovery --query spectra.mgf --database DNP_unique.sdf --output identifications.tsv
```

## Evaluation signals

- Top-1 accuracy on 3,964 negative-mode GNPS spectra against unique DNP with the default (unretrained) model should be ~36%
- PNNL lipid library vs. LMSD search should reproduce top-1 = 16.9%, top-3 = 37.4%, top-5 = 47.6%, top-10 = 63.0%
- A search of over 8 million GNPS spectra should yield ~3,185 unique small molecules identified at 0% FDR, a six-fold increase over previous methods
- Every query spectrum must have its candidates ranked by molDiscovery matching score before accuracy computation
- Write the four lipid benchmark accuracy values to lipid_benchmark_accuracies.csv and compare to reported values

## Limitations

- Default pretrained model was trained on combined CID/HCD positive-mode spectra; searching negative-mode spectra without retraining drops accuracy to 36%
- Lower accuracy in low mass ranges (<600 Da), partially because fewer fragments are generated for small molecules
- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules; benzene rings and resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by instrument type, dissociation technique, ionization mode, and collision energy, which are not fully modeled
- No p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules

## Evidence

- [abstract] mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] searched against unique DNP: "dictionary of natural products (DNP)"
- [discussion] probabilistic model learned from GNPS spectral library: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [intro] identified 3185 unique small molecules at 0% FDR: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [discussion] default model trained on positive-mode CID/HCD: "fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [readme] README: web test interface contact: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
