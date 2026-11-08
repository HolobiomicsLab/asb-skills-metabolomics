---
name: large-scale-spectrum-pair-dataset-verification
description: Use when an agent has embedded an independent MS/MS dataset (e.g. mzML
  files through `gleams embed`) and must validate the derived positive/negative spectrum
  pair counts against published figures before using them for evaluation or training.
when_to_use_negative:
- When the goal is to train GLEAMS on MassIVE-KB v1 data rather than verify pair statistics
  of an independent evaluation dataset — training uses 30 million PSMs compiled from
  MassIVE metadata TSVs, not pair-count verification.
- 'When spectra have not yet been embedded: pair generation presupposes completed
  `gleams embed` output and PSM identifications; applying this check to raw vendor
  files or unconverted data is premature.'
- When no PSM identifications (peptide labels) are available for the dataset, since
  positive/negative labeling depends entirely on same-vs-different peptide sequence
  assignments.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Generates the 32-dimensional embeddings of MS/MS spectra via `gleams embed`,
    which are the basis for candidate pair construction and verification; model weights
    loaded from gleams_82c0124b.hdf5 (v0.3 release).
  repo: https://github.com/bittremieux/GLEAMS
- name: ProteoWizard msconvert
  role: Converts vendor raw mass spectrometry files (e.g. from PRIDE PXD015943) to
    mzML peak files that `gleams embed` can read.
- name: Python
  role: Used to recompute and verify pair counts (positive pairs with identical peptide
    sequences; negative pairs within 10 ppm precursor mass tolerance) from embeddings
    metadata and PSM labels.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:76c09658b2877a7046ce39e7e5ed0971da8ac33362d31634def3031ffd3647db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - ProteoWizard msconvert
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Large-scale spectrum pair dataset verification

## Summary

Verifies that a large-scale spectrum pair dataset generated from GLEAMS embeddings (e.g. from an independent phosphoproteomics dataset like PXD015943) matches the reported scale and labeling logic: positives are PSM pairs with identical peptide sequences, negatives are distinct-peptide pairs within a precursor mass tolerance.

## When to use

Use when an agent has embedded an independent MS/MS dataset (e.g. mzML files through `gleams embed`) and must validate the derived positive/negative spectrum pair counts against published figures before using them for evaluation or training.

## When NOT to use

- When the goal is to train GLEAMS on MassIVE-KB v1 data rather than verify pair statistics of an independent evaluation dataset — training uses 30 million PSMs compiled from MassIVE metadata TSVs, not pair-count verification.
- When spectra have not yet been embedded: pair generation presupposes completed `gleams embed` output and PSM identifications; applying this check to raw vendor files or unconverted data is premature.
- When no PSM identifications (peptide labels) are available for the dataset, since positive/negative labeling depends entirely on same-vs-different peptide sequence assignments.

## Inputs

- mzML peak files from an independent dataset (e.g. PRIDE PXD015943 phosphoproteomics raw files converted via ProteoWizard msconvert)
- 32-dimensional GLEAMS embeddings (NumPy .npy array) with corresponding metadata (GLEAMS_embed.parquet)
- PSM identifications (e.g. Mascot) at 1% FDR with peptide sequence and precursor mass annotations

## Outputs

- Verified counts of embedded spectra, PSMs, positive pairs, and negative pairs (expected: 18.6M spectra, ~3.7M PSMs, 1.185B positive pairs, 293M negative pairs for PXD015943)
- Labeled spectrum pair list (positive = same peptide sequence; negative = different peptide, precursor within 10 ppm)

## How to apply

After running `gleams embed` on converted mzML peak files to obtain 32-dimensional embeddings, use the available Mascot PSM identifications (at 1% FDR) to label candidate spectrum pairs: pairs of PSMs sharing the same peptide sequence are positive, while pairs with different peptide sequences and precursor masses within 10 ppm are negative. For the PXD015943 phosphoproteomics reference run, the expected verification targets are 18.6 million embedded spectra, ~3.7 million PSMs at 1% FDR, 1.185 billion positive pairs, and 293 million negative pairs. Verify counts by recomputing pair cardinality from the peptide-label groups and the precursor mass tolerance filter, and cross-check that the number of embedded spectra in the output parquet metadata matches the input mzML spectra count. The 10 ppm precursor tolerance bounds which negative pairs are admissible, so an incorrect tolerance is the most likely source of count discrepancy.

## Related tools

- **GLEAMS** (Generates the 32-dimensional embeddings of MS/MS spectra via `gleams embed`, which are the basis for candidate pair construction and verification; model weights loaded from gleams_82c0124b.hdf5 (v0.3 release).) — https://github.com/bittremieux/GLEAMS
- **ProteoWizard msconvert** (Converts vendor raw mass spectrometry files (e.g. from PRIDE PXD015943) to mzML peak files that `gleams embed` can read.)
- **Python** (Used to recompute and verify pair counts (positive pairs with identical peptide sequences; negative pairs within 10 ppm precursor mass tolerance) from embeddings metadata and PSM labels.)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Embedded spectrum count matches the expected scale (18.6 million spectra for PXD015943) as recorded in GLEAMS_embed.parquet metadata.
- PSM count at 1% FDR is approximately 3.7 million for the PXD015943 phosphoproteomics dataset.
- Positive pair count equals 1.185 billion when pairs are defined as PSMs with identical peptide sequences.
- Negative pair count equals 293 million when pairs are restricted to PSMs with different peptide sequences within a 10 ppm precursor mass tolerance.
- Row count of the embedding .npy array (n x 32) equals the number of MS/MS spectra read from the input mzML files.

## Limitations

- Verification targets are dataset-specific: the 18.6M spectra / 3.7M PSMs / 1.185B positive / 293M negative figures apply to PXD015943 with Mascot PSMs; other datasets will yield different counts.
- Pair labeling depends on the quality of PSM identifications; misassigned peptide labels directly distort positive/negative pair proportions.
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU, and Git LFS bandwidth limits may block straightforward installation of model weights.

## Evidence

- [other] After embedding the phosphoproteomics spectra, 1.185 billion positive pairs (PSMs with identical peptide sequences) and 293 million negative pairs (PSMs with different sequences within a 10ppm precursor mass tolerance) were generated.: [evidence span withheld — anchor status: not_found]
- [readme] GLEAMS provides the `gleams embed` command to convert MS/MS spectra in peak files to 32-dimensional embeddings.: [evidence span withheld — anchor status: external_doc]
- [readme] Manually download the GLEAMS model weights from the latest release.: "Manually download the [GLEAMS model weights from the latest release](https://github.com/bittremieux/GLEAMS/releases/download/v0.3/gleams_82c0124b.hdf5)"
- [intro] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [methods] All 185 million PSMs at 1% FDR obtained using MSGF+: "All 185 million PSMs at 1% FDR obtained using MSGF+"
