---
name: proteomics-psm-label-pairing
description: Use when spectra have PSM peptide identifications (e.g., Mascot or MSGF+
  results at 1% FDR) and you need positive/negative spectrum pairs to train or evaluate
  a spectral embedding on an independent dataset such as the phosphoproteomics PXD015943
  study.
when_to_use_negative:
- Spectra lack trusted PSM peptide identifications, so no ground-truth labels exist
  to assign positive/negative pairs (use unsupervised gleams cluster instead).
- The goal is direct hierarchical clustering of already-embedded spectra (--distance_threshold
  0.3), not pair generation for contrastive training or evaluation.
- Input data are not tandem MS/MS spectra with precursor mass information, so the
  10 ppm negative-pair constraint cannot be applied.
edam_operation: http://edamontology.org/operation_3631
tools:
- name: GLEAMS
  role: Embeds MS/MS spectra into the 32-dimensional space whose pairwise distances
    the positive/negative PSM pairs are meant to constrain
  repo: https://github.com/bittremieux/GLEAMS
- name: MSGF+
  role: Source of PSM peptide identifications at 1% FDR used to derive pair labels
    from the MassIVE-KB training data
- name: Python
  role: Implementation environment (Python 3.8) for building candidate pairs and labeling
    them from PSM metadata
- name: conda
  role: Environment setup for the GLEAMS installation used to embed spectra before
    pairing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:642135407bd040a27944cc81698bd8e595c8cdfa0c6f4588360a91dfad040e98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - MSGF+
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3631
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# proteomics-psm-label-pairing

## Summary

Generates labeled spectrum pairs from peptide-spectrum match (PSM) identifications to train or evaluate a spectral embedding model such as GLEAMS: pairs of spectra from PSMs with identical peptide sequences are positive pairs, while PSMs with different peptide sequences but close precursor mass are negative pairs.

## When to use

Use when spectra have PSM peptide identifications (e.g., Mascot or MSGF+ results at 1% FDR) and you need positive/negative spectrum pairs to train or evaluate a spectral embedding on an independent dataset such as the phosphoproteomics PXD015943 study.

## When NOT to use

- Spectra lack trusted PSM peptide identifications, so no ground-truth labels exist to assign positive/negative pairs (use unsupervised gleams cluster instead).
- The goal is direct hierarchical clustering of already-embedded spectra (--distance_threshold 0.3), not pair generation for contrastive training or evaluation.
- Input data are not tandem MS/MS spectra with precursor mass information, so the 10 ppm negative-pair constraint cannot be applied.

## Inputs

- mzML peak files of MS/MS spectra (e.g. converted from PRIDE PXD015943 raw files with ProteoWizard msconvert)
- PSM identifications with peptide sequence labels (Mascot PSMs for PXD015943; MSGF+ PSMs at 1% FDR for MassIVE-KB training data)
- Trained GLEAMS model weights (gleams_82c0124b.hdf5 from the v0.3 release)

## Outputs

- Positive spectrum pairs: PSMs sharing identical peptide sequences (e.g. 1.185 billion pairs for PXD015943)
- Negative spectrum pairs: PSMs with different peptide sequences within a 10 ppm precursor mass tolerance (e.g. 293 million pairs for PXD015943)
- 32-dimensional embeddings (n x 32 NumPy array, e.g. GLEAMS_embed.npy) with corresponding metadata (GLEAMS_embed.parquet)

## How to apply

First convert vendor raw files to mzML and embed all MS/MS spectra with the trained GLEAMS model (v0.3 weights gleams_82c0124b.hdf5) to obtain 32-dimensional embeddings, e.g. 18.6 million embedded spectra for PXD015943. Then build candidate spectrum pairs restricted to PSMs identified at 1% FDR (about 3.7 million PSMs in the Hijazi phosphoproteomics case). Label a pair positive when both PSMs carry the identical peptide sequence, and negative when the peptides differ but the precursor masses lie within a 10 ppm tolerance, so negatives are hard cases rather than trivially separable mass mismatches. The same pairing scheme underlies training, where 30 million PSMs from the MassIVE-KB (v1) dataset supplied the labeled pairs. Verify pair statistics against reported scale (e.g. 1.185 billion positive and 293 million negative pairs for PXD015943) as a correctness check.

## Related tools

- **GLEAMS** (Embeds MS/MS spectra into the 32-dimensional space whose pairwise distances the positive/negative PSM pairs are meant to constrain) — https://github.com/bittremieux/GLEAMS
- **MSGF+** (Source of PSM peptide identifications at 1% FDR used to derive pair labels from the MassIVE-KB training data)
- **Python** (Implementation environment (Python 3.8) for building candidate pairs and labeling them from PSM metadata)
- **conda** (Environment setup for the GLEAMS installation used to embed spectra before pairing)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Positive pairs consist exclusively of PSMs with identical peptide sequences; negative pairs have distinct peptides but precursor masses within 10 ppm.
- For the PXD015943 evaluation, reproduced counts match the reported scale: 18.6 million embedded spectra, ~3.7 million PSMs at 1% FDR, 1.185 billion positive pairs, and 293 million negative pairs.
- Embedded output is a two-dimensional NumPy array of dimension n x 32 with a matching parquet metadata file for the embedded spectra.
- Only PSMs at 1% FDR enter pair generation, mirroring the MassIVE-KB identification filtering used for training.

## Limitations

- Negative pairs are only meaningful within a 10 ppm precursor mass tolerance; outside it, pairs are trivially separable by mass and add little training signal.
- Pair counts scale combinatorially with the number of PSMs (billions of pairs from ~3.7 million PSMs), requiring sampling or efficient candidate generation.
- Labels depend entirely on PSM correctness: identifications above the 1% FDR threshold would inject mislabeled pairs.

## Evidence

- [readme] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [methods] All 185 million PSMs at 1% FDR obtained using MSGF+: "All 185 million PSMs at 1% FDR obtained using MSGF+"
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
- [readme] These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors).: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource (maximum top 100 PSMs for 2.1 million unique precursors)."
