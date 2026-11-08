---
name: peptide-spectrum-match-training-data-compilation
description: Use when a neural spectral-embedding model (e.g. GLEAMS) needs a labeled
  peptide-spectrum-match training corpus and no pre-packaged training set exists,
  requiring assembly from MassIVE-KB library metadata and raw spectra retrieval.
when_to_use_negative:
- You only need embeddings or clustering of existing spectra — use `gleams embed`/`gleams
  cluster` instead; no training-data compilation is required
- Your spectra are not Human HCD data from MassIVE-KB; the compiled library is organism-
  and acquisition-specific, so a different labeled corpus is needed
- You lack the storage/bandwidth to fetch spectra from the MassIVE FTP server; the
  30M PSM corpus is very large and not available as a single download
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GLEAMS
  role: Target model whose training data is compiled; training configuration is set
    in gleams/config.py after the corpus is assembled
  repo: https://github.com/bittremieux/GLEAMS
- name: conda
  role: Creates the Python 3.8 environment with compiler tools and GPU runtime needed
    to train on the compiled dataset
- name: MSGF+
  role: Search engine whose 185 million PSMs at 1% FDR form the superset from which
    the MassIVE-KB training PSMs derive
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:899db43179b29a6823c287f482ffdaaec90f606c217bd06b1199d00f8ea1b616
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - MSGF+
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# peptide-spectrum-match-training-data-compilation

## Summary

Compile a large-scale PSM training corpus by downloading the MassIVE-KB (v1) Human HCD candidate library spectra metadata TSV (30 million PSMs, max top 100 PSMs per unique precursor) and retrieving the corresponding peak files from the MassIVE FTP server, extracting spectra by scan number. This provides the supervised training data required to train the GLEAMS Siamese encoder.

## When to use

Use when a neural spectral-embedding model (e.g. GLEAMS) needs a labeled peptide-spectrum-match training corpus and no pre-packaged training set exists, requiring assembly from MassIVE-KB library metadata and raw spectra retrieval.

## When NOT to use

- You only need embeddings or clustering of existing spectra — use `gleams embed`/`gleams cluster` instead; no training-data compilation is required
- Your spectra are not Human HCD data from MassIVE-KB; the compiled library is organism- and acquisition-specific, so a different labeled corpus is needed
- You lack the storage/bandwidth to fetch spectra from the MassIVE FTP server; the 30M PSM corpus is very large and not available as a single download

## Inputs

- MassIVE-KB (v1) Human HCD Spectral Library 'All Candidate library spectra' metadata TSV (zipped; columns include 'filename' and 'scan')
- MS/MS peak files (mzML) retrieved from the MassIVE FTP server
- Proteosafe_task search task identifiers (for optional full 185M PSM mzTab retrieval)

## Outputs

- Compiled training dataset of 30 million PSMs (subset of MassIVE-KB, max top 100 PSMs for 2.1 million unique precursors)
- Paired metadata (peptide identification, charge, filename, scan) and extracted MS/MS spectra ready for GLEAMS training via gleams/config.py

## How to apply

Navigate the MassIVE website to MassIVE Knowledge Base > Human HCD Spectral Library (task 82c0124b) > All Candidate library spectra > Download, yielding a zipped TSV with metadata and peptide identifications for all 30 million PSMs. Because the spectra cannot be downloaded as a single archive, use the 'filename' column of the TSV to fetch matching peak files from the MassIVE FTP server and extract individual spectra by their 'scan' column value. Optionally retrieve the full 185 million PSMs at 1% FDR (obtained with MSGF+, per Wang et al.) by extracting unique 'proteosafe_task' identifiers from the metadata TSV and downloading mzTab files from per-task ProteoSAFe result pages. The training subset is defined as maximum top 100 PSMs for 2.1 million unique precursors; once compiled, configure training via gleams/config.py in a Python 3.8 Conda environment with a CUDA-enabled GPU.

## Related tools

- **GLEAMS** (Target model whose training data is compiled; training configuration is set in gleams/config.py after the corpus is assembled) — https://github.com/bittremieux/GLEAMS
- **conda** (Creates the Python 3.8 environment with compiler tools and GPU runtime needed to train on the compiled dataset)
- **MSGF+** (Search engine whose 185 million PSMs at 1% FDR form the superset from which the MassIVE-KB training PSMs derive)

## Examples

```
conda env create -f https://raw.githubusercontent.com/bittremieux/GLEAMS/master/environment.yml && conda activate gleams && pip install git+https://github.com/bittremieux/GLEAMS.git
```

## Evaluation signals

- Metadata TSV row count matches the expected ~30 million PSMs from MassIVE-KB (v1) Human HCD candidate library spectra
- Every training PSM resolves to a real peak file via its 'filename' column and a valid scan number via its 'scan' column on the MassIVE FTP server
- Precursor diversity check: the corpus covers 2.1 million unique precursors with a maximum of top 100 PSMs per precursor
- Training converges under a Conda environment with a CUDA-enabled GPU using gleams/config.py, with validation loss recorded on a held-out split
- Downstream ablation (removing precursor/fragment/reference-spectrum feature blocks) reproduces the Extended Data Fig. 9 ordering of validation loss

## Limitations

- The 30M PSM training spectra 'are not readily available as a single download' — compilation requires per-filename FTP retrieval and per-scan extraction, which is laborious and bandwidth-heavy
- Git LFS bandwidth limits on the GLEAMS repo may cause 'This repository is over its data quota' errors; weights must be manually downloaded (gleams_82c0124b.hdf5) and placed in data/
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU, so compilation/training cannot proceed on unsupported platforms

## Evidence

- [readme] GLEAMS was trained on 30 million PSMs from the MassIVE-KB (v1) dataset: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset](https://massive.ucsd.edu/ProteoSAFe/static/massive-kb-libraries.jsp)."
- [readme] Compilation procedure via MassIVE-KB candidate library spectra download: "go to [MassIVE Knowledge Base](https://massive.ucsd.edu/ProteoSAFe/static/massive-kb-libraries.jsp) > [Human HCD Spectral Library](https://massive.ucsd.edu/ProteoSAFe/status.jsp?task=82c0124b6053407fa"
- [readme] Spectra retrieved per filename and scan number from MassIVE FTP: [evidence span withheld — anchor status: not_found]
- [readme] Training subset definition: max top 100 PSMs for 2.1 million unique precursors: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors)."
- [methods] 185 million PSMs at 1% FDR obtained with MSGF+ available as mzTab: "All 185 million PSMs at 1% FDR obtained using MSGF+"
- [readme] Environment requirements for training: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
