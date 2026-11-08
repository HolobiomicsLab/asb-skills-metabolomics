---
name: fragmentation-tree-computation
description: Use when you have high-resolution MS/MS spectra of small molecules and
  need to go beyond elemental composition — computing fragment formulas and fragmentation
  events to rank molecular formulas or feed downstream structure identification.
when_to_use_negative:
- Input is already a formula-annotated fragmentation tree or the user only needs database
  lookup of known formulas — the tree computation adds no information
- Spectra are low-resolution or derived from compounds outside the 'small molecules
  of biological interest' scope; SIRIUS 4 targets LC-MS/MS data of metabolites and
  small molecules
- The task is structure identification alone with a trusted formula; in that case
  use CSI:FingerID directly without recomputing the tree
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS 4
  role: Runs the fragmentation tree computation on MS/MS peak lists, integrating isotope
    pattern analysis to rank molecular formulas
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Receives computed fragmentation trees and spectra uploaded from SIRIUS to
    rank candidate structures in molecular structure databases
- name: SIRIUS
  role: General SIRIUS framework (GUI and CLI) that hosts the fragmentation tree computation
    tool
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:fceb9a2383b76a7b4b5969a24bcb99563b5863a996c529a47cd19df831a16ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fragmentation-tree-computation

## Summary

SIRIUS's fragmentation tree computation interprets tandem mass spectra by inferring a hypothetical fragmentation tree in which nodes are annotated with fragment molecular formulas and arcs represent fragmentation losses. This tree computation underlies SIRIUS 4's rapid molecular formula ranking and feeds CSI:FingerID structure searches, enabling identification rates above 70% on challenging metabolomics datasets.

## When to use

Use when you have high-resolution MS/MS spectra of small molecules and need to go beyond elemental composition — computing fragment formulas and fragmentation events to rank molecular formulas or feed downstream structure identification.

## When NOT to use

- Input is already a formula-annotated fragmentation tree or the user only needs database lookup of known formulas — the tree computation adds no information
- Spectra are low-resolution or derived from compounds outside the 'small molecules of biological interest' scope; SIRIUS 4 targets LC-MS/MS data of metabolites and small molecules
- The task is structure identification alone with a trusted formula; in that case use CSI:FingerID directly without recomputing the tree

## Inputs

- High-resolution tandem mass spectra (MS/MS peak lists with m/z and abundance) for small compounds
- Optional isotope pattern / MS1 data used to decompose the parent ion
- Element alphabet specification (e.g. CHNOPS or CHNOPS+BrClFI)
- Ionization mode (positive or negative)

## Outputs

- Hypothetical fragmentation tree with nodes annotated with molecular formulas of fragments and arcs representing fragmentation events (losses)
- Ranked candidate molecular formulas for the precursor ion
- Fragmentation trees and spectra suitable for upload to CSI:FingerID for structure search

## How to apply

Preprocess the tandem mass spectra so the spectrometer output is a list of peaks with masses and abundances, then run SIRIUS's fragmentation tree computation to analyze the fragmentation pattern and infer a hypothetical fragmentation tree with formula-annotated fragment nodes and loss-annotated edges. Provide the isotope pattern (from MS1 data if available) alongside the MS/MS peaks so the parent ion can be decomposed and the correct molecular formula hypothesis anchored; SIRIUS uses the parent peak from MS1 to decompose the parent ion when MS1 data is available. Specify the element alphabet (e.g. CHNOPS only, or CHNOPS with BrClFI when bromine, chlorine, fluorine or iodine may be present) and the ionization mode (positive or negative), since formula ranking depends on these choices. The computed fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID web service to rank candidate structures, which is the main downstream consumer of the tree output. Benchmark context: on the 1,553-compound CHNOPS benchmark, SIRIUS 4 computes fragmentation trees 332-fold faster than SIRIUS 3.0; verify your run finishes within comparable wall-clock limits before feeding results forward.

## Related tools

- **SIRIUS 4** (Runs the fragmentation tree computation on MS/MS peak lists, integrating isotope pattern analysis to rank molecular formulas) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Receives computed fragmentation trees and spectra uploaded from SIRIUS to rank candidate structures in molecular structure databases)
- **SIRIUS** (General SIRIUS framework (GUI and CLI) that hosts the fragmentation tree computation tool) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input spectra.ms --formula --tree --output results/ && curl -X POST https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/search --data @results/tree.ms
```

## Evaluation signals

- The computed tree's root fragment formula matches the precursor mass within the instrument's mass accuracy (element decomposition consistency)
- Fragment nodes form a connected acyclic structure where each edge loss corresponds to a chemically plausible neutral loss
- Formula ranking: for CASMI 2016 reanalysis, correct formula expected at top 1 for 93.8% of compounds using MS/MS + isotope pattern, 96.2% at top 2
- Wall-clock runtime: for a 1,553-compound CHNOPS benchmark dataset, SIRIUS 4 must finish dramatically faster (332-fold) than the SIRIUS 3.0 baseline
- Result output from the command line tool retains the final version structure (no deprecated fields) before upload to CSI:FingerID

## Limitations

- Manual interpretation of tandem mass spectra is time-consuming and non-trivial — the tree is a hypothesis, not ground truth
- Isotope pattern analysis was rewritten and is 'hopefully more stable than before', so instability in isotope pattern selection may affect tree quality
- Bugs were reported in penalizing molecular formulas on intrinsically charged mode and in isotope pattern selection, which can skew formula ranking feeding the tree
- Support for in-source fragmentations and adducts is experimental
- Command-line result output structure reached its final version only in later releases, so older outputs may differ

## Evidence

- [methods] tree inference method: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments and arcs (edges) represent"
- [methods] throughput capability: "SIRIUS allows for the automated and high-throughput analysis of small-compound MS data beyond elemental composition without requiring compound structures or a mass spectral database."
- [methods] isotope input: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [discussion] parent ion handling: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the parent ion"
- [methods] downstream integration: "Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services."
- [abstract] validation rate: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [readme] readme tree description: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments and arcs (edges) represent"
