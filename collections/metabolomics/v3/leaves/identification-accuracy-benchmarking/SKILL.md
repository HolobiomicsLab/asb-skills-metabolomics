---
name: identification-accuracy-benchmarking
description: Use when you need to quantify how accurately a database-search method
  (molDiscovery) identifies small molecules from labeled reference spectra, especially
  to probe performance gaps like negative-ion-mode spectra searched with the default
  positive-mode-trained model.
when_to_use_negative:
- Query spectra are positive-mode CID/HCD spectra, which match the default model's
  training conditions — the benchmark would not probe the ionization-mode generalization
  gap
- The goal is discovering novel molecules absent from the structure database rather
  than measuring accuracy against annotated reference spectra
- The fragmentation model has already been retrained on the query ionization mode,
  making the ~36% negative-mode baseline inapplicable as a validation target
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Mass spectral database search tool that scores molecule-spectrum pairs with
    a probabilistic fragmentation model; its top-ranked candidate is compared to known
    GNPS annotations to compute accuracy
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:fd165656fc29a1ec46bbe3e01f4713bacc15733df34210d573bd86b1cf4c6204
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# identification-accuracy-benchmarking

## Summary

Benchmarks molDiscovery's small molecule identification accuracy by searching reference spectra (e.g., 3,964 negative-mode spectra from the GNPS spectral library) against a non-redundant structure database such as the Dictionary of Natural Products (DNP), computing top-1 accuracy without retraining the default model.

## When to use

Use when you need to quantify how accurately a database-search method (molDiscovery) identifies small molecules from labeled reference spectra, especially to probe performance gaps like negative-ion-mode spectra searched with the default positive-mode-trained model.

## When NOT to use

- Query spectra are positive-mode CID/HCD spectra, which match the default model's training conditions — the benchmark would not probe the ionization-mode generalization gap
- The goal is discovering novel molecules absent from the structure database rather than measuring accuracy against annotated reference spectra
- The fragmentation model has already been retrained on the query ionization mode, making the ~36% negative-mode baseline inapplicable as a validation target

## Inputs

- 3,964 negatively charged reference spectra from the GNPS spectral library with known compound annotations
- Non-redundant Dictionary of Natural Products (DNP) molecular structure database
- molDiscovery default pretrained probabilistic fragmentation model (trained on combined CID/HCD positive-mode spectra)

## Outputs

- Top-ranked candidate structure per query spectrum
- Top-1 identification accuracy (fraction of spectra with correct top-scoring candidate, ~36% expected for negative mode without retraining)

## How to apply

Obtain labeled reference spectra (e.g., 3,964 negatively charged spectra from the GNPS spectral library) and a structure database (the non-redundant DNP). Run molDiscovery's database search using the default pretrained probabilistic fragmentation model, which was trained on combined CID/HCD positive-mode spectra, without retraining on the query mode. For each query spectrum, record the top-ranked DNP candidate structure and compare it to the known compound identity from the GNPS library annotation. Compute top-1 accuracy as the fraction of spectra whose top-scoring candidate matches the known structure; the expected baseline is ~36% for the unmodified negative-mode setup.

## Related tools

- **molDiscovery** (Mass spectral database search tool that scores molecule-spectrum pairs with a probabilistic fragmentation model; its top-ranked candidate is compared to known GNPS annotations to compute accuracy) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- Computed top-1 accuracy on the 3,964 negative-mode GNPS spectra against non-redundant DNP should be ~36% when using the unmodified default model
- Every query spectrum must have exactly one recorded top-ranked DNP candidate to allow per-spectrum matching against the GNPS library annotation
- Accuracy denominator must equal the number of searched query spectra (3,964); the database searched must be the unique (non-redundant) DNP
- As a positive reference, on 8 million GNPS tandem mass spectra molDiscovery identified 3185 unique small molecules at 0% FDR, a six times increase over prior methods
- Confirm no retraining or fine-tuning was applied to the fragmentation model during the benchmark run

## Limitations

- The default model was trained on combined CID/HCD positive-mode spectra, and fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled, degrading negative-mode accuracy
- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are treated as alternating single/double bonds
- Accuracy is lower in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules
- No p-value computation exists for general molecules; the MCMC p-value approach applies only to peptidic molecules

## Evidence

- [results] Without retraining on negative spectra, molDiscovery reached 36% accuracy when tested on 3964 negatively charged spectra of the GNPS spectral library: [evidence span withheld — anchor status: not_found]
- [discussion] Mass spectrometry fragmentation is influenced by experimental factors: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [intro] molDiscovery identified 3185 unique small molecules at 0% false discovery rate: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [abstract] Here, we present molDiscovery, a mass spectral database search method: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [readme] To test molDiscovery for academic users, please visit: "To test molDiscovery for academic users, please visit : "
