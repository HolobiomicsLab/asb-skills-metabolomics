---
name: lc-ms2-feature-annotation
description: Use when a crude natural product or metabolomics extract has been acquired
  by LC-MS2 and you need to prioritize which molecular features to fractionate or
  isolate based on spectral similarity and dereplication.
when_to_use_negative:
- Input is already a purified, structurally solved compound — no fractionation prioritization
  is needed, use direct dereplication instead
- MS2 spectra are absent or of too low quality for cosine-similarity edge propagation
  — use a purely NMR-based (SMART 2.0) or target-list approach
- The study goal is not isolation but, e.g., global spectral statistics — network-based
  prioritization adds unnecessary overhead
tools:
- name: molecular networking
  role: Propagates MS2 cosine similarity edges across the feature list to cluster
    structurally related nodes and surface prioritization targets
- name: SMART 2.0
  role: Provides NMR-based mixture-class recognition that is used alongside the MS2
    network to guide fractionation of the Symploca extract
- name: NP Classifier
  role: API for programmatic classification of SMILES strings into natural-product
    classes, used in the downstream annotation of the same dataset (mwang87/NP-Classifier)
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:1e239c13f13be1ff39f7b9051f704e45c13c1572ab7cfced594c3d174606c822
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# LC-MS2 feature annotation

## Summary

Prioritize and annotate LC-MS2 features from a crude extract by propagating MS2 spectral similarity across a molecular network, then confirm top nodes with 1D/2D NMR and LC-MS2 dereplication. In this study it guided cytotoxicity-driven fractionation of a marine cyanobacterium (Symploca) extract to symplocolide A and swinholide/samholide derivatives.

## When to use

Use when a crude natural product or metabolomics extract has been acquired by LC-MS2 and you need to prioritize which molecular features to fractionate or isolate based on spectral similarity and dereplication.

## When NOT to use

- Input is already a purified, structurally solved compound — no fractionation prioritization is needed, use direct dereplication instead
- MS2 spectra are absent or of too low quality for cosine-similarity edge propagation — use a purely NMR-based (SMART 2.0) or target-list approach
- The study goal is not isolation but, e.g., global spectral statistics — network-based prioritization adds unnecessary overhead

## Inputs

- LC-MS2 peak list / feature table of the crude extract (m/z, RT, MS2 spectra)
- MS2 spectra of library or dereplication matches
- Fraction-level cytotoxicity data (cancer cell assays)
- Isolated compound 1D/2D NMR data for structural confirmation

## Outputs

- Molecular network nodes/edges table (e.g. molecular_network.tsv) with MS2 cosine edge scores
- Prioritized list of molecular features to guide fractionation
- Structural annotations (swinholide A, samholides A–I, symplocolide A) with supporting NMR/LC-MS2 evidence

## How to apply

Load the LC-MS2 peak list of the Symploca extract into a molecular networking workflow, run spectral library matching, and propagate edge scores via MS2 cosine similarity so that structurally related nodes cluster together. Combine the resulting network with the cytotoxicity profile of fractions and with NMR-based SMART 2.0 mixture analysis to decide which nodes to chase during fractionation. Isolate the prioritized compound (here symplocolide A) and confirm its planar structure by 1D/2D NMR and LC-MS2, then annotate the surrounding network nodes by library/dereplication matches to swinholide A and samholides A–I. The rationale is that MS2 cosine edges give a similarity scaffold that lets a few dereplicated anchors (e.g. known swinholide-class macrolides) pull dozens of unannotated neighbors into a chemically coherent cluster.

## Related tools

- **molecular networking** (Propagates MS2 cosine similarity edges across the feature list to cluster structurally related nodes and surface prioritization targets)
- **SMART 2.0** (Provides NMR-based mixture-class recognition that is used alongside the MS2 network to guide fractionation of the Symploca extract)
- **NP Classifier** (API for programmatic classification of SMILES strings into natural-product classes, used in the downstream annotation of the same dataset (mwang87/NP-Classifier)) — https://github.com/mwang87/NP-Classifier

## Evaluation signals

- Edges in the exported network table should carry MS2 cosine similarity scores that cluster known anchors (e.g. swinholide A) with unannotated neighbors
- The prioritized feature list should correspond to nodes whose fractionation is supported by both cytotoxicity and SMART 2.0 evidence, not by network alone
- Isolated compound (symplocolide A) should be structurally confirmed by 1D/2D NMR and LC-MS2 as a swinholide/luminaolide hybrid
- Network neighbors should reproduce the reported dereplication outcome — i.e. nodes matching swinholide A and samholides A–I derivatives

## Limitations

- Network clustering depends on MS2 cosine similarity; features with noisy or missing MS2 spectra may not cluster and can be missed
- The approach guides prioritization but does not solve planar structure on its own — confirmation still requires 1D/2D NMR and LC-MS2
- Requires paired cytotoxicity or NMR-class evidence to avoid chasing network nodes that are not biologically or chemically relevant

## Evidence

- [abstract] MS2-based molecular networking was applied, together with cytotoxicity and SMART 2.0, to prioritize fractionation of the Symploca extract, leading to isolation and rapid identification of symplocolide A and annotation of swinholide A, samholides A–I, and several new derivatives.: [evidence span withheld — anchor status: not_found]
- [abstract] Marine cyanobacterium Symploca extract was roughly fractionated and prioritized by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking.: [evidence span withheld — anchor status: not_found]
- [methods] 1. Load the LC-MS2 peak list of the Symploca extract. 2. Run spectral library matching and propagate edge scores via the MS2 cosine similarity used in GNPS molecular networking.: [evidence span withheld — anchor status: not_found]
