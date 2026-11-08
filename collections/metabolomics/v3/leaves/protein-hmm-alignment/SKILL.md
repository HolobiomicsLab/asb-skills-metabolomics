---
name: protein-hmm-alignment
description: Use when you need to assign Pfam-domain architecture to protein sequences
  — e.g., aligning representative BGC proteins against biosynthetic domain pHMMs before
  hierarchical clade clustering or feature-vector construction.
when_to_use_negative:
- Input is already a Pfam-domain annotation table or BGC feature matrix — skip straight
  to feature-vectorization/clustering rather than re-running pHMM alignment
- You need pairwise BGC comparison for GCA network analysis (BiG-SCAPE territory),
  not domain-level homology detection
- Sequences are nucleotide or whole-genome contigs needing BGC prediction first (antiSMASH/ClusterFinder/DeepBGC)
  rather than protein-vs-pHMM scanning
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: BiG-SLiCE
  role: provides the 293-core-biosynthetic-domain pHMM database (download_bigslice_hmmdb)
    and consumes the alignment-derived sub-Pfam feature set
  repo: https://github.com/medema-group/bigslice
- name: PyHMMER
  role: Cython bindings to HMMER3 used by BiG-SLiCE v2 for the pHMM scanning step,
    replacing the standalone HMMER suite
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:f26072b596aaa90cf5928c86d9d995d4d50d497a18ba1169971d9017459e5428
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - PyHMMER
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# protein-hmm-alignment

## Summary

Align protein sequences against profile HMMs (e.g., the 293 core biosynthetic domain pHMMs used by BiG-SLiCE) with HMMER/PyHMMER to detect Pfam-domain homology, the foundation for building sub-Pfam (signature domain fingerprinting) features.

## When to use

Use when you need to assign Pfam-domain architecture to protein sequences — e.g., aligning representative BGC proteins against biosynthetic domain pHMMs before hierarchical clade clustering or feature-vector construction.

## When NOT to use

- Input is already a Pfam-domain annotation table or BGC feature matrix — skip straight to feature-vectorization/clustering rather than re-running pHMM alignment
- You need pairwise BGC comparison for GCA network analysis (BiG-SCAPE territory), not domain-level homology detection
- Sequences are nucleotide or whole-genome contigs needing BGC prediction first (antiSMASH/ClusterFinder/DeepBGC) rather than protein-vs-pHMM scanning

## Inputs

- FASTA of representative protein sequences (e.g., 9,451,490 RP15 / Release 2020_01 proteins)
- Profile HMM database (293 core biosynthetic domain pHMMs from the BiG-SLiCE HMM database, bigslice-models-2022-11-30)

## Outputs

- Per-sequence domain hit list (domain architecture assignments per protein)
- Aligned sequences per core domain, grouped into hierarchical clades (10–100 clades per domain)
- subpfam.hmm — 3,889 sublevel pHMM models (Supplementary Table S2)

## How to apply

Retrieve the pHMM database with `download_bigslice_hmmdb` (bigslice-models-2022-11-30) and verify the Biosynthetic-pfam md5 (37495cac452bf1dd8aff2c4ad92065fe). Run the representative RP15 protein sequences (9,451,490 in the paper) against the 293 preselected core biosynthetic domain pHMMs using HMMER (or PyHMMER in BiG-SLiCE v2, which replaced the standalone HMMER suite and can be pip-installed). Collect per-sequence domain hits, which become the input for hierarchical clade clustering per core domain. Retain only domains resolving into 10–100 clades, build a sublevel pHMM from each clade alignment, and package the models into subpfam.hmm, checking it against the Sub-pfam md5 (2e6b41d06f3c318c61dffb022798091e). The rationale: generic biosynthetic-Pfam models miss sequence-level diversity within core domains, so sub-Pfam fingerprinting captures finer granularity.

## Related tools

- **BiG-SLiCE** (provides the 293-core-biosynthetic-domain pHMM database (download_bigslice_hmmdb) and consumes the alignment-derived sub-Pfam feature set) — https://github.com/medema-group/bigslice
- **PyHMMER** (Cython bindings to HMMER3 used by BiG-SLiCE v2 for the pHMM scanning step, replacing the standalone HMMER suite) — https://github.com/althonos/pyhmmer

## Examples

```
mamba env create -n bigslice -c conda-forge -c bioconda bigslice; download_bigslice_hmmdb
```

## Evaluation signals

- Rebuilt sub-Pfam library contains exactly 3,889 sub-Pfam models matching Supplementary Table S2
- Sub-pfam md5 equals 2e6b41d06f3c318c61dffb022798091e
- Biosynthetic-pfam md5 of the downloaded HMM database equals 37495cac452bf1dd8aff2c4ad92065fe
- Each retained core domain resolves into 10–100 hierarchical clades; domains outside this range are excluded
- Hit set restricted to the 293 preselected core biosynthetic domain pHMMs

## Limitations

- BiG-SLiCE v2.0 updated pHMM databases to PFAM 35.0 and switched to PyHMMER, so v2 model outputs may differ from the original v1-era 3,889 sub-Pfam library built for the paper
- Domains whose aligned sequences resolve into fewer than 10 or more than 100 clades are dropped, so alignment coverage does not guarantee a sub-Pfam model for every core domain
- PyHMMER follows semantic versioning but its API changed substantially before v0.10; pin versions for reproducibility

## Evidence

- [readme] PyHMMER for the alignment step: "Switching from HMMER to [pyHMMER](https://github.com/althonos/pyhmmer) (__speed-ups__, can now be fully installed via __pip__)"
- [readme] PyHMMER scope covers the workflows used here: [evidence span withheld — anchor status: external_doc]
- [supplementary] Alignment against 293 core biosynthetic domain pHMMs precedes clade clustering: [evidence span withheld — anchor status: not_found]
- [readme] PyHMMER is Cython bindings over HMMER3, the engine actually doing the alignment: [evidence span withheld — anchor status: external_doc]
