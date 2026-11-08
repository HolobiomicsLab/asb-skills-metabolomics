---
name: hierarchical-protein-sequence-clustering
description: 'Use when you need finer-than-domain resolution for BGC feature vectors:
  input is a set of protein sequences (e.g., RP15 representatives) and a curated pHMM
  library of core biosynthetic domains, and generic Pfam-level features are too coarse
  for GCF clustering.'
when_to_use_negative:
- The goal is whole-BGC or GCF-level clustering at distance thresholds (e.g., BiG-SLiCE
  T = 300–1,500) rather than sub-domain sequence-resolution fingerprinting
- Input is nucleotide sequence rather than protein sequences — pHMM alignment requires
  translated protein inputs
- Only one protein per domain is available; hierarchical clade discovery requires
  multi-sequence diversity per domain
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: BiG-SLiCE
  role: Distributes the Biosynthetic-pfam (293 core domains) and Sub-pfam (3,889 models)
    pHMM databases and defines the clade-acceptance range (10–100 clades per core
    domain) used to build sub-Pfam features
  repo: https://github.com/medema-group/bigslice
- name: PyHMMER
  role: Python/Cython bindings used for protein-sequence pHMM scanning (and the toolchain
    used in place of standalone HMMER in BiG-SLiCE 2.0) when aligning representatives
    against core domain HMMs
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:b7083faadedba04632c20932512f0e4d48a6b6d588fbacef05a3973afa80eeee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - PyHMMER
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hierarchical-protein-sequence-clustering

## Summary

Builds a fine-grained sub-Pfam signature-domain fingerprint library by hierarchically clustering protein sequences already annotated with core biosynthetic domain pHMMs into clades and deriving a sublevel pHMM from each clade. In BiG-SLiCE this converts 293 coarse core biosynthetic Pfam domains into 3,889 sub-Pfam models that capture sequence-level diversity the generic chemical-class-level models miss.

## When to use

Use when you need finer-than-domain resolution for BGC feature vectors: input is a set of protein sequences (e.g., RP15 representatives) and a curated pHMM library of core biosynthetic domains, and generic Pfam-level features are too coarse for GCF clustering.

## When NOT to use

- The goal is whole-BGC or GCF-level clustering at distance thresholds (e.g., BiG-SLiCE T = 300–1,500) rather than sub-domain sequence-resolution fingerprinting
- Input is nucleotide sequence rather than protein sequences — pHMM alignment requires translated protein inputs
- Only one protein per domain is available; hierarchical clade discovery requires multi-sequence diversity per domain

## Inputs

- Representative protein sequences (e.g., 9,451,490 RP15 Release 2020_01 sequences, FASTA)
- Core biosynthetic domain pHMM library (293 Biosynthetic-pfam HMMs, bigslice-models-2022-11-30 release)
- HMMER/PyHMMER toolchain

## Outputs

- subpfam.hmm — a pHMM library of 3,889 sub-Pfam signature models (Sub-pfam md5: 2e6b41d06f3c318c61dffb022798091e)
- Per-core-domain clade assignments of aligned protein sequences
- Model inventory matching Supplementary Table S2

## How to apply

First obtain the reference HMM database (e.g., via the BiG-SLiCE command download_bigslice_hmmdb, release bigslice-models-2022-11-30) and verify the Biosynthetic-pfam md5 checksum 37495cac452bf1dd8aff2c4ad92065fe. Align the representative protein sequences (the 9,451,490 RP15 release-2020_01 sequences in the reference study) against the 293 preselected core biosynthetic domain pHMMs using HMMER/PyHMMER. For each core domain, hierarchically cluster the aligned protein sequences into clades, and accept only domains that resolve into 10–100 clades — this range filters out domains that are either too conserved or too fragmented to yield informative submodels. Build a sublevel pHMM from the multiple alignment of each accepted clade's sequences, producing in the reference study 3,889 sub-Pfam models (Supplementary Table S2), packaged as subpfam.hmm.

## Related tools

- **BiG-SLiCE** (Distributes the Biosynthetic-pfam (293 core domains) and Sub-pfam (3,889 models) pHMM databases and defines the clade-acceptance range (10–100 clades per core domain) used to build sub-Pfam features) — https://github.com/medema-group/bigslice
- **PyHMMER** (Python/Cython bindings used for protein-sequence pHMM scanning (and the toolchain used in place of standalone HMMER in BiG-SLiCE 2.0) when aligning representatives against core domain HMMs) — https://github.com/althonos/pyhmmer

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice && mamba activate bigslice && download_bigslice_hmmdb
```

## Evaluation signals

- The rebuilt library contains exactly 3,889 sub-Pfam models whose identities match Supplementary Table S2
- subpfam.hmmd5 equals 2e6b41d06f3c318c61dffb022798091e
- Biosynthetic-pfam md5 equals 37495cac452bf1dd8aff2c4ad92065fe before alignment
- Every accepted core domain resolves into 10–100 hierarchical clades; domains outside this range are dropped
- Each sublevel pHMM is built from the multiple alignment of a single accepted clade's sequences

## Limitations

- Only core domains resolving into 10–100 clades are retained, so domains that are too conserved or too diverse contribute no sub-Pfam models
- Checksum validation is release-specific (bigslice-models-2022-11-30); BiG-SLiCE 2.0 updated the pHMM databases to PFAM 35.0, so rebuilt models will differ from older releases
- Model counts depend on the input representative set (RP15 Release 2020_01); a different sequence set will change clade structure and model count

## Evidence

- [methods] The sub-Pfam feature set is built by aligning representative protein sequences against 293 core biosynthetic domain pHMMs, hierarchically clustering the aligned sequences into clades: [evidence span withheld — anchor status: not_found]
- [supplementary] Build a sublevel pHMM from the alignment of each accepted clade, producing a total of 3,889 sub-Pfam models (Supplementary Table S2).: [evidence span withheld — anchor status: not_found]
- [readme] Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__): "Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__)"
- [readme] pHMM databases have been updated to __PFAM 35.0__: "pHMM databases have been updated to __PFAM 35.0__"
