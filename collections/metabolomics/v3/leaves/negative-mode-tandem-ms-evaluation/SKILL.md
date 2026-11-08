---
name: negative-mode-tandem-ms-evaluation
description: Use when benchmarking a small molecule database search tool (e.g., molDiscovery)
  on negative-ion-mode tandem mass spectra against a structure database such as DNP,
  especially to quantify the penalty of using a positive-mode-trained fragmentation
  model.
when_to_use_negative:
- Evaluating positive-mode spectra or a model retrained on negative-mode spectra —
  the ~36% figure applies only to the unretrained default model on negative-mode spectra
- Spectral library search (searching against reference spectra) rather than in silico
  search against molecular structure databases like DNP
- Comparing against baselines on datasets where molDiscovery's reported advantage
  (e.g., 3,185 unique molecules at 0% FDR on 8 million GNPS spectra) was established
  in positive mode
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Database search tool whose default probabilistic fragmentation model is evaluated
    on negative-mode spectra against the non-redundant DNP database without retraining
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:4a533bf37203da3b7921c435b6d3621d887f5a0def49a1dd96442f80e1317779
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# negative-mode-tandem-ms-evaluation

## Summary

Benchmark protocol for evaluating how well a mass spectral database search tool identifies small molecules from negatively charged tandem MS spectra without retraining its fragmentation model. Grounded in molDiscovery's evaluation on 3,964 negative-mode GNPS spectral library spectra searched against the non-redundant Dictionary of Natural Products (DNP), which yielded ~36% top-1 accuracy.

## When to use

Use when benchmarking a small molecule database search tool (e.g., molDiscovery) on negative-ion-mode tandem mass spectra against a structure database such as DNP, especially to quantify the penalty of using a positive-mode-trained fragmentation model.

## When NOT to use

- Evaluating positive-mode spectra or a model retrained on negative-mode spectra — the ~36% figure applies only to the unretrained default model on negative-mode spectra
- Spectral library search (searching against reference spectra) rather than in silico search against molecular structure databases like DNP
- Comparing against baselines on datasets where molDiscovery's reported advantage (e.g., 3,185 unique molecules at 0% FDR on 8 million GNPS spectra) was established in positive mode

## Inputs

- 3,964 negatively charged reference spectra from the GNPS spectral library
- Known structure annotations from the GNPS spectral library (ground-truth compound identities)
- Non-redundant Dictionary of Natural Products (DNP) structure database

## Outputs

- Top-ranked DNP candidate structure per query spectrum
- Top-1 identification accuracy (expected ~36% with the default positive-mode-trained molDiscovery model)

## How to apply

Obtain the negative-mode reference spectra from the GNPS spectral library (3,964 negatively charged spectra in the paper) and the non-redundant Dictionary of Natural Products (DNP) structure database. Run molDiscovery's database search using its default pretrained probabilistic fragmentation model — trained on combined CID/HCD positive-mode spectra — without retraining on negative spectra, since fragmentation depends on ionization mode. For each query spectrum, record the top-ranked DNP candidate and compare it to the known structure annotation from the GNPS library. Compute top-1 accuracy as the fraction of query spectra whose top-scoring candidate matches the known compound; the paper reports ~36% in this setting, which serves as the validation target. Interpret the relatively low accuracy as evidence that the default model does not capture negative-mode fragmentation behavior, motivating mode-specific retraining.

## Related tools

- **molDiscovery** (Database search tool whose default probabilistic fragmentation model is evaluated on negative-mode spectra against the non-redundant DNP database without retraining) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- Computed top-1 accuracy on the 3,964 negative-mode GNPS spectra searched against unique DNP should be approximately 36%
- Every query spectrum must return a top-ranked candidate from the non-redundant DNP database
- Accuracy must be computed as exact structure-identity matches against the GNPS library annotations, not relaxed mass-only matches
- Verify the search used the default pretrained model (combined CID/HCD positive-mode training) with no negative-mode retraining
- Results should be reproducible using the mohimanilab/molDiscovery repository

## Limitations

- The default model was trained on combined CID/HCD positive-mode spectra, so fragmentation is not fully modeled for other ionization modes, instrument types, dissociation techniques, or collision energies
- Lower accuracy is observed in low mass ranges (<600 Da), partly because fewer fragments are generated for molecules in this range
- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds

## Evidence

- [discussion] fragmentation depends on ionization mode; default model trained on combined CID/HCD positive-mode spectra: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [abstract] molDiscovery is a mass spectral database search method for small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [discussion] molDiscovery learned a probabilistic model from GNPS spectral library reference spectra: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [intro] The DNP is one of the molecular structure databases molDiscovery can search: "dictionary of natural products (DNP)"
- [readme] The tool is available for academic testing via a web server and by contacting the authors: "To test molDiscovery for academic users, please visit :"
