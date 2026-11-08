---
name: greedy-transitive-spectral-alignment
description: Use when a molecular network component contains multiple MS/MS spectra
  and you need to compute which peaks correspond across all of them, beyond pairwise
  matching.
when_to_use_negative:
- Input is a single pair of spectra — modified cosine alone suffices; MMSA is designed
  for alignment across multiple spectra in a component.
- Smaller peak alignments (e.g. y2 peaks in the albicidin family) are not fully captured
  by the greedy multiple alignment, so use a different method if every minor shared
  peak must be recovered.
- The analysis targets shifted peaks like the C12 HAA (m/z 239.162) or rhamnose-bound-C12
  (m/z 385.219) that MMSA did not capture in the rhamnolipid case.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: 'Implements the multiple mass spectral alignment pipeline: filtering, scoring,
    MST construction, transitive alignment, and JSON serialization'
- name: NetworkX 3.3
  role: Constructs the minimum spanning tree over spectra, with edges weighted by
    pairwise modified cosine similarity scores subtracted from 1
- name: GNPS2
  role: Supplies molecular network components (task ID and component number) as input
    to the alignment
- name: NetworkFamily_MultipleAlignment_Website
  role: Web-based application exposing the alignment via GNPS Task ID, USI, or FBMN
    input methods; provides interactive visualization and SVG export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:7d7a9b0da827615ff4eccdc1fe72fe36985a145bb16fe54dbcc4ecb6c39f7898
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkFamily_MultipleAlignment_Website
  - NetworkX
  - NetworkX 3.3
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Greedy transitive spectral alignment

## Summary

Builds a minimum spanning tree over MS/MS spectra weighted by modified cosine similarity and performs transitive, greedy alignment of peaks across all spectra in a molecular network component, identifying consensus peak sets.

## When to use

Use when a molecular network component contains multiple MS/MS spectra and you need to compute which peaks correspond across all of them, beyond pairwise matching.

## When NOT to use

- Input is a single pair of spectra — modified cosine alone suffices; MMSA is designed for alignment across multiple spectra in a component.
- Smaller peak alignments (e.g. y2 peaks in the albicidin family) are not fully captured by the greedy multiple alignment, so use a different method if every minor shared peak must be recovered.
- The analysis targets shifted peaks like the C12 HAA (m/z 239.162) or rhamnose-bound-C12 (m/z 385.219) that MMSA did not capture in the rhamnolipid case.

## Inputs

- MS/MS spectra (peak lists with m/z and intensity values) from a molecular network component
- GNPS2 task ID and component number, or a set of Universal Spectrum Identifiers (mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>), or FBMN task ID and component numbers
- Preprocessed spectrum objects with filtered peaks (top 6 within ±25 m/z, excluding ±17 m/z of precursor), sqrt-transformed and normalized intensities

## Outputs

- Aligned peak sets, each annotated with m/z and intensity values across the component's spectra
- MST structure connecting the spectra in the component, serialized to JSON
- Sets of consensus peaks that drive molecular networking formation and highlight chemical modifications or core structures

## How to apply

Preprocess each MS/MS spectrum by filtering to the top 6 most intense peaks within a ±25 m/z window, excluding peaks within ±17 m/z of the precursor ion, then apply a square-root transformation to intensities and normalize them. Score all spectrum pairs using the modified cosine algorithm (_cosine_fast) to produce a similarity matrix. Convert pairwise scores to edge weights by subtracting each score from 1 and build a minimum spanning tree over the spectra using NetworkX 3.3's greedy MST approach. Apply topological sorting to process MST nodes in a specific order and perform transitive alignment of spectra pairs along the tree using _cosine_fast to obtain peak matches. Finally, compute the non-transitively aligned peak sets after the transitive alignment step and serialize the resulting MST structure and aligned peak sets (each annotated with m/z and intensity values) to a JSON file.

## Related tools

- **MMSA** (Implements the multiple mass spectral alignment pipeline: filtering, scoring, MST construction, transitive alignment, and JSON serialization)
- **NetworkX 3.3** (Constructs the minimum spanning tree over spectra, with edges weighted by pairwise modified cosine similarity scores subtracted from 1)
- **GNPS2** (Supplies molecular network components (task ID and component number) as input to the alignment)
- **NetworkFamily_MultipleAlignment_Website** (Web-based application exposing the alignment via GNPS Task ID, USI, or FBMN input methods; provides interactive visualization and SVG export)

## Examples

```
python main.py  # runs the Flask/Dash app; enter a GNPS Task ID + component number, then click 'Process and Save JSON' to produce the aligned-peak-sets JSON
```

## Evaluation signals

- The output JSON file exists and parses successfully.
- The JSON contains an MST/alignment structure connecting all spectra in the component.
- Each aligned peak set is annotated with m/z and intensity values for each spectrum in the component.
- Peak filtering preserved exactly the top 6 most intense peaks within ±25 m/z, excluding peaks within ±17 m/z of the precursor ion.
- Edge weights in the MST equal 1 minus the pairwise modified cosine similarity score for each spectrum pair.

## Limitations

- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7) in the rhamnolipid family.
- Smaller peak alignments such as y2 peaks in the albicidin family are not fully captured in the greedy multiple alignment.
- The greedy MST-based approach may miss some transitive peak correspondences that a global optimal alignment would find.

## Evidence
