---
name: ms2-molecular-networking-annotation
description: Use when you have LC-MS2 peak lists from a fractionated natural product
  extract and need to prioritize fractions for isolation or compare unknowns against
  reference compounds (e.g., swinholide A) via spectral similarity networks.
when_to_use_negative:
- Input is a purified single-compound spectrum — spectral library matching alone suffices
  and networking adds nothing
- Data lacks MS2 fragmentation (e.g., 1D NMR or MS1-only data) — use SMART 2.0 NMR-based
  mixture analysis instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: molecular networking
  role: MS2-based molecular networking to prioritize fractionation and annotate network
    nodes against the swinholide A reference
- name: atomic sort
  role: Method applied for candidate ranking when comparing network nodes against
    the swinholide A reference
- name: SMART 2.0
  role: Complementary NMR-based mixture analysis used together with molecular networking
    and cytotoxicity to prioritize fractions
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:862012700536d78ba0a4c817d44695428b0f8e766ab285110b8431d9784c0e83
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SMART 2.0
  - atomic sort
  - docker
  - docker-compose
  - molecular networking
  - tensorflow
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# ms2-molecular-networking-annotation

## Summary

MS2-based molecular networking for prioritizing and annotating natural products in LC-MS2 data of crude/fractionated extracts, used here alongside cytotoxicity and SMART 2.0 NMR analysis to guide isolation from an environmental Symploca extract.

## When to use

Use when you have LC-MS2 peak lists from a fractionated natural product extract and need to prioritize fractions for isolation or compare unknowns against reference compounds (e.g., swinholide A) via spectral similarity networks.

## When NOT to use

- Input is a purified single-compound spectrum — spectral library matching alone suffices and networking adds nothing
- Data lacks MS2 fragmentation (e.g., 1D NMR or MS1-only data) — use SMART 2.0 NMR-based mixture analysis instead

## Inputs

- LC-MS2 peak list of the fractionated Symploca extract
- LC-MS2 data of prioritized fractions
- Reference spectra for comparison (e.g., swinholide A)
- Fraction cytotoxicity priorities (cancer cell data, for prioritization)

## Outputs

- Nodes/edges tables of the molecular network (molecular_network.tsv)
- Molecular network with nodes annotated against reference compounds (e.g., swinholide A, samholides A–I)
- Fraction-prioritization results guiding isolation

## How to apply

Load the LC-MS2 peak list of the fractionated extract and run spectral library matching, propagating edge scores via the MS2 cosine similarity used in GNPS molecular networking. Export the resulting network as nodes/edges tables (molecular_network.tsv). Generate MS2-based molecular networking from the prioritized fractions and compare network nodes against reference compounds such as swinholide A, applying the 'atomic sort' method for candidate ranking. In this study, networking was combined with cancer cell cytotoxicity data and NMR-based SMART 2.0 mixture analysis to prioritize fractionation of the Symploca extract, leading to isolation of the chimeric swinholide-like macrolide symplocolide A and annotation of swinholide A, samholides A–I, and several new derivatives.

## Related tools

- **molecular networking** (MS2-based molecular networking to prioritize fractionation and annotate network nodes against the swinholide A reference)
- **atomic sort** (Method applied for candidate ranking when comparing network nodes against the swinholide A reference)
- **SMART 2.0** (Complementary NMR-based mixture analysis used together with molecular networking and cytotoxicity to prioritize fractions) — github.com/mwang87/NP-Classifier

## Evaluation signals

- Exported molecular_network.tsv contains both nodes and edges tables with edge scores propagated via MS2 cosine similarity
- Known reference compounds (e.g., swinholide A) are recovered as annotated nodes in the network
- Output reproduces the reported annotation of symplocolide A as a new chimeric swinholide-like macrolide alongside swinholide A and samholides A–I
- Prioritized fractions correspond to those guided by cancer cell cytotoxicity and SMART 2.0 NMR annotations
- Planar-structure assignments are corroborated by independent 1D/2D NMR and LC-MS2 evidence

## Limitations

- Molecular networking is an annotation tool — planar structure confirmation of symplocolide A required subsequent 1D/2D NMR and LC-MS2 analysis
- Network annotation does not assign stereochemistry; only planar structure of the chimeric macrolide was established
- Prioritization is strongest when combined with orthogonal signals (cytotoxicity, NMR-based SMART 2.0), not networking alone

## Evidence

- [abstract] MS2-based molecular networking was applied, together with cytotoxicity and SMART 2.0, to prioritize fractionation of the Symploca extract: [evidence span withheld — anchor status: not_found]
- [methods] Load the LC-MS2 peak list of the Symploca extract. Run spectral library matching and propagate edge scores via the MS2 cosine similarity used in GNPS molecular networking. Export the network as nodes/edges tables (molecular_network.tsv).: [evidence span withheld — anchor status: not_found]
- [methods] compare network nodes against the swinholide A reference, applying the 'atomic sort' method for candidate ranking: [evidence span withheld — anchor status: not_found]
