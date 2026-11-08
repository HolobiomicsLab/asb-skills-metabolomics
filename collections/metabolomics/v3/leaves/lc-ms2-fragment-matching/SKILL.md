---
name: lc-ms2-fragment-matching
description: Use when LC-MS2 data from fractionated natural-product extracts must
  be dereplicated against a reference compound (e.g. swinholide A) to prioritize fractions
  and annotate known or structural-hybrid macrolides.
when_to_use_negative:
- Input is 1H NMR spectra of crude mixtures rather than LC-MS2 data — use SMART 2.0
  NMR-based mixture analysis instead
- Goal is programmatic compound-class assignment from SMILES strings — use the NP
  Classifier /classify API instead
- Input is already a curated feature table with confirmed compound identities; fragment
  matching would only duplicate existing dereplication
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: molecular networking
  role: MS2-based molecular networking clusters LC-MS2 fragmentation of prioritized
    fractions so nodes can be compared to the swinholide A reference for dereplication
    and analogue discovery
- name: atomic sort
  role: Candidate-ranking method applied to molecular network nodes to prioritize
    which clusters/nodes to pursue for isolation
- name: SMART 2.0
  role: Complementary NMR-based machine-learning annotation used alongside MS2 molecular
    networking to prioritize fractions before fragment matching
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:bd8350d7051f7d5c1c97ca21374c14ae2bb3501189ad554bc841668d34c0ab42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - SMART 2.0
  - atomic sort
  - docker
  - docker-compose
  - molecular networking
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lc-ms2-fragment-matching

## Summary

Generates MS2-based molecular networks from LC-MS2 data of prioritized natural-product fractions and compares network nodes against reference compounds (e.g. swinholide A) to annotate known and chimeric macrolides. In this study it guided the isolation and planar-structure confirmation of symplocolide A alongside 1D/2D NMR.

## When to use

Use when LC-MS2 data from fractionated natural-product extracts must be dereplicated against a reference compound (e.g. swinholide A) to prioritize fractions and annotate known or structural-hybrid macrolides.

## When NOT to use

- Input is 1H NMR spectra of crude mixtures rather than LC-MS2 data — use SMART 2.0 NMR-based mixture analysis instead
- Goal is programmatic compound-class assignment from SMILES strings — use the NP Classifier /classify API instead
- Input is already a curated feature table with confirmed compound identities; fragment matching would only duplicate existing dereplication

## Inputs

- LC-MS2 data (mzML or equivalent) of fractionated environmental Symploca extract fractions
- Fraction prioritization metadata (cancer cell cytotoxicity ranks, SMART 2.0 NMR annotations)
- Reference compound MS2 spectrum(s), e.g. swinholide A

## Outputs

- MS2-based molecular network with nodes annotated against the reference compound
- Candidate ranking of network nodes via the atomic sort method
- Planar structure assignment for the isolated macrolide (symplocolide A) validated by 1D/2D NMR and LC-MS2

## How to apply

First roughly fractionate the environmental Symploca extract and prioritize fractions using cancer cell cytotoxicity together with NMR-based SMART 2.0 annotations. Then generate MS2-based molecular networking from the LC-MS2 data of the prioritized fractions, and compare network nodes against a reference compound spectrum — here swinholide A — applying the recently appearing 'atomic sort' method for candidate ranking. Nodes matching the reference are annotated as known molecules (swinholide A, samholides A–I and derivatives), while orphan nodes with related fragmentation are flagged as candidate new analogues. Candidate structures are then confirmed by matching 1D/2D NMR and LC-MS2 evidence, which established symplocolide A as a structural hybrid between swinholide A and luminaolide B. Rationale: molecular networking clusters similar fragmentation patterns, so a swinholide-like node adjacent to known swinholides signals a related but potentially novel macrolide worth isolating.

## Related tools

- **molecular networking** (MS2-based molecular networking clusters LC-MS2 fragmentation of prioritized fractions so nodes can be compared to the swinholide A reference for dereplication and analogue discovery)
- **atomic sort** (Candidate-ranking method applied to molecular network nodes to prioritize which clusters/nodes to pursue for isolation)
- **SMART 2.0** (Complementary NMR-based machine-learning annotation used alongside MS2 molecular networking to prioritize fractions before fragment matching) — https://github.com/mwang87/NP-Classifier

## Evaluation signals

- Network nodes from prioritized fractions match the swinholide A reference node in the MS2-based molecular network
- Known molecules are annotated as expected: swinholide A, samholides A–I, and several new derivatives
- The prioritized orphan node resolves to symplocolide A, confirmed as a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS2
- The fraction-prioritization table, SMART 2.0 annotation results, and structure assignment compile into a single consistent record
- Candidate ranking is reproducible when the atomic sort method is re-applied to the same network

## Limitations

- MS2 molecular networking alone yields planar-level similarity, not stereostructure; 1D/2D NMR confirmation is required for the planar structure assignment
- Guided fractionation depends on biological prioritization (cancer cell cytotoxicity) in addition to spectral data; purely spectral matching may miss low-abundance analogues
- Ranking of candidates depends on the specific 'atomic sort' method recently introduced, which may not be available in all molecular networking deployments

## Evidence
