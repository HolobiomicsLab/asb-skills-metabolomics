---
name: gcms-ei-spectral-edge-analysis
description: Use when you have EI molecular networking output (edges table and annotated
  spectra/consensus spectra) from deconvoluted GC-MS data and need to test whether
  cosine-similarity edges reflect structural similarity based on chemical class, despite
  the molecular ion often being absent from EI spectra.
when_to_use_negative:
- Data are LC-MS/MS spectra with true MS2 fragmentation and intact precursor masses
  — standard MolNetEnhancer workflows in pyMolNetEnhancer (classical or feature-based,
  e.g. with MS2LDA Mass2Motifs) apply directly instead of the GC-MS EI edge-propagation
  adaptation
- The analysis goal is deconvolution of co-eluting GC-MS peaks rather than interpretation
  of an already-deconvoluted molecular network — use the MSHub auto-deconvolution
  (unsupervised non-negative matrix factorization) workflow instead
- Every node already carries a confident direct chemical class annotation (e.g. molecular
  ion present and library-matched), making edge-wise propagation of class labels unnecessary
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0153
tools:
- name: pyMolNetEnhancer
  role: GC-MS-adapted notebook used to propagate chemical structural annotations across
    EI molecular network edges and label each consensus spectrum with its chemical
    class
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform whose molecular networking outputs (edges and annotated spectra tables)
    for GC-MS data are the inputs analyzed; workflows for molecular networking of
    GC-MS data run within GNPS
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: GNPS_GC (MSHub analysis)
  role: Companion repository of the manuscript that releases the EI molecular networking
    output files loaded into the Jupyter environment for this analysis
  repo: github.com/bittremieux/GNPS_GC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:6e48d64a335aba1b828f2cae2aee5147c0220a0c2cd8afb8386b7b1f6e7096f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - GNPS
  - GNPS_GC (MSHub analysis)
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gcms-ei-spectral-edge-analysis

## Summary

Skill for determining what drives the organization of EI-based cosine similarity molecular networks built from deconvoluted GC-MS data, by propagating chemical class annotations across network edges with a GC-MS-adapted pyMolNetEnhancer workflow and quantifying within-class versus between-class edge fractions.

## When to use

Use when you have EI molecular networking output (edges table and annotated spectra/consensus spectra) from deconvoluted GC-MS data and need to test whether cosine-similarity edges reflect structural similarity based on chemical class, despite the molecular ion often being absent from EI spectra.

## When NOT to use

- Data are LC-MS/MS spectra with true MS2 fragmentation and intact precursor masses — standard MolNetEnhancer workflows in pyMolNetEnhancer (classical or feature-based, e.g. with MS2LDA Mass2Motifs) apply directly instead of the GC-MS EI edge-propagation adaptation
- The analysis goal is deconvolution of co-eluting GC-MS peaks rather than interpretation of an already-deconvoluted molecular network — use the MSHub auto-deconvolution (unsupervised non-negative matrix factorization) workflow instead
- Every node already carries a confident direct chemical class annotation (e.g. molecular ion present and library-matched), making edge-wise propagation of class labels unnecessary

## Inputs

- EI molecular networking edges table (GNPS output) from deconvoluted GC-MS data
- Annotated spectra / consensus spectra table with chemical class annotations
- GC-MS-adapted MolNetEnhancer (pyMolNetEnhancer) notebook

## Outputs

- figure_s7a_edge_similarity.png (edge-structural-similarity plot)
- Per-edge chemical class annotation table (within-class vs between-class edge labels)
- Distribution of within-class versus between-class edges

## How to apply

Load the EI molecular networking output files (edges and annotated spectra tables) from the released GNPS_GC companion repository (github.com/bittremieux/GNPS_GC) into a Jupyter environment. Run the GC-MS-adapted pyMolNetEnhancer notebook to propagate chemical structural annotations across molecular network edges and label each consensus spectrum with its chemical class. For each EI cosine-similarity edge, annotate whether the connected nodes belong to the same chemical class, then compute the distribution of within-class versus between-class edges. Visualize this distribution as the edge-structural-similarity plot corresponding to Figure S7a, and save the figure as figure_s7a_edge_similarity.png together with the per-edge class annotation table. The rationale is that because EI spectra frequently lack the molecular ion, node-level chemical identities cannot be assigned directly, so structural similarity must be inferred edge-wise from propagated chemical class annotations.

## Related tools

- **pyMolNetEnhancer** (GC-MS-adapted notebook used to propagate chemical structural annotations across EI molecular network edges and label each consensus spectrum with its chemical class) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform whose molecular networking outputs (edges and annotated spectra tables) for GC-MS data are the inputs analyzed; workflows for molecular networking of GC-MS data run within GNPS) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **GNPS_GC (MSHub analysis)** (Companion repository of the manuscript that releases the EI molecular networking output files loaded into the Jupyter environment for this analysis) — github.com/bittremieux/GNPS_GC

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- The large majority of EI cosine-similarity edges connect compounds sharing the same chemical class (within-class edges dominate over between-class edges)
- The reproduced edge-structural-similarity figure qualitatively matches Figure S7a of the paper upon expert review against the published panel
- The per-edge annotation table assigns a chemical class to both endpoints of every EI cosine-similarity edge
- Outputs are produced as required: figure_s7a_edge_similarity.png plus the accompanying per-edge class annotation table
- The pipeline ran end-to-end on the released GNPS_GC repository outputs without unannotated edges remaining

## Limitations

- The molecular ion is often absent from EI spectra, so chemical class labels cannot be assigned to nodes directly and must be inferred by propagation across network edges
- Validation of the reproduced Figure S7a is qualitative (expert review against the published panel) rather than backed by a quantitative metric
- The stock pyMolNetEnhancer README describes MS2LDA Mass2Motif mapping for MS/MS molecular networks (classical and feature-based GNPS workflows), not EI GC-MS data; adapting it to EI edges is paper-specific
- Per the pyMolNetEnhancer README, large MS2LDA-related downloads may hit server connection timeouts, so files may need to be downloaded manually from ms2lda.org

## Evidence

- [supplementary] EI-based cosine similarity molecular networks are predominantly driven by structural similarity based on chemical class annotations, as shown by network analysis of EI data in Figure S7a.: [evidence span withheld — anchor status: not_found]
- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks: [evidence span withheld — anchor status: external_doc]
