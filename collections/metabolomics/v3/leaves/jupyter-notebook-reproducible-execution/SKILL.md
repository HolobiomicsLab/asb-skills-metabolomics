---
name: jupyter-notebook-reproducible-execution
description: Use when a companion repository (e.g. bittremieux/GNPS_GC) provides example
  notebooks and workflow output files (edges, annotated spectra tables), and the goal
  is to re-run the analysis to validate a published figure or table.
when_to_use_negative:
- Input is raw GC-MS vendor data that has not yet been deconvoluted and networked
  through the MSHub/GNPS workflow — run deconvolution and molecular networking first
  to obtain edges and spectra tables
- The analysis concerns MS2 feature-based molecular networking with MS2LDA motif mapping
  rather than EI-based cosine-similarity networks of GC-MS consensus spectra
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Python module executed in Jupyter notebooks to map chemical class and substructure
    information onto GNPS molecular networks
  repo: github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform whose molecular networking output files (edges, spectra, job IDs)
    serve as notebook inputs
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:cd2c9c37bd2f5342c2ec713659046b4eedbae6a015d8bb8b8a99baa995470fdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# jupyter-notebook-reproducible-execution

## Summary

Reproduce published GC-MS molecular-networking analyses (e.g. Figure S7a edge-structural-similarity plots) by executing companion Jupyter notebooks (pyMolNetEnhancer) against released GNPS_GC output files, then comparing generated figures and tables against the paper's panels.

## When to use

Use when a companion repository (e.g. bittremieux/GNPS_GC) provides example notebooks and workflow output files (edges, annotated spectra tables), and the goal is to re-run the analysis to validate a published figure or table.

## When NOT to use

- Input is raw GC-MS vendor data that has not yet been deconvoluted and networked through the MSHub/GNPS workflow — run deconvolution and molecular networking first to obtain edges and spectra tables
- The analysis concerns MS2 feature-based molecular networking with MS2LDA motif mapping rather than EI-based cosine-similarity networks of GC-MS consensus spectra

## Inputs

- EI molecular networking edges output file from a GNPS GC-MS job
- Annotated consensus spectra table (nodes with chemical class annotations)
- GNPS job ID / MGF clustered spectra downloaded from GNPS
- Example notebook from the companion repository (e.g. pyMolNetEnhancer Example_notebooks/*.ipynb)

## Outputs

- Reproduced figure (e.g. figure_s7a_edge_similarity.png) matching the published panel
- Per-edge chemical-class annotation table (within-class vs between-class edges)
- GraphML/TSV network outputs importable into Cytoscape

## How to apply

Clone the companion repository (github.com/bittremieux/GNPS_GC) and load its released EI molecular networking outputs (edges file and annotated consensus spectra table) into a Jupyter environment. Install the notebook dependency with `pip install pyMolNetEnhancer`, then run the GC-MS-adapted pyMolNetEnhancer notebook line by line as the README instructs ('execute the code ... line by line'), specifying the GNPS job ID and user-defined mapping parameters. Propagate chemical class annotations across the EI cosine-similarity network edges, label each consensus spectrum with its chemical class, and for each edge record whether the two connected nodes share the same chemical class. Compute the within-class versus between-class edge distribution and export both the figure (e.g. figure_s7a_edge_similarity.png) and the per-edge class annotation table. The rationale is that pyMolNetEnhancer integrates chemical class information into GNPS molecular networks, and reproducible execution of the notebook against the same input files is the validation mechanism for findings such as EI networks being predominantly driven by chemical-class structural similarity.

## Related tools

- **pyMolNetEnhancer** (Python module executed in Jupyter notebooks to map chemical class and substructure information onto GNPS molecular networks) — github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform whose molecular networking output files (edges, spectra, job IDs) serve as notebook inputs) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
pip install pyMolNetEnhancer && jupyter notebook Example_notebooks/ChemicalClasses_2_Network.ipynb
```

## Evaluation signals

- Reproduced edge-structural-similarity figure qualitatively matches Figure S7a of the paper: the large majority of EI cosine-similarity edges connect compounds sharing the same chemical class
- Notebook executes end-to-end without errors after `pip install pyMolNetEnhancer`, with all specified IDs/parameters supplied
- Output table contains a class-annotation column for every network edge (within-class vs between-class label)
- Generated GraphML/TSV files load into Cytoscape with node/edge tables matching the notebook's documented columns (e.g. CLUSTERID1/CLUSTERID2, interaction)

## Limitations

- Server connection timeouts can occur when downloading large MS2LDA/GNPS files; the README advises downloading the file manually from ms2lda.org as a fallback
- Probability and overlap thresholds for motif mapping must be set consistently (defaults: prob=0.01, overlap=0.3); the summary table already contains filtered relations using thresholds set in the web app
- Reproduction is validated qualitatively (expert review against the published panel), not by an automated numeric equality check

## Evidence

- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Product Social Molecular Networking (GNPS) platform.: [evidence span withheld — anchor status: not_found]
- [readme] Install pyMolNetEnhancer with: `pip install pyMolNetEnhancer`: [evidence span withheld — anchor status: external_doc]
- [supplementary] EI-based cosine similarity molecular networks are predominantly driven by structural similarity based on chemical class annotations, as shown by network analysis of EI data in Figure S7a.: [evidence span withheld — anchor status: not_found]
