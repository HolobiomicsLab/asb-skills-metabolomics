---
name: consensus-peak-set-identification
description: Use when a molecular family (e.g. ~80 rhamnolipid or albicidin MS/MS
  spectra from GNPS2) needs a global peak alignment that explains which consensus
  peaks cause spectra to cluster, rather than only pairwise cosine scores.
when_to_use_negative:
- Input is a pairwise spectral comparison or a molecular-network edge score only —
  MMSA is specifically for the alignment of entire molecular families, not pairwise
  visualization
- The analysis requires exact capture of every small peak alignment (e.g. y2 fragments
  in albicidin) — the greedy multiple alignment does not fully capture smaller peak
  sets
- The spectra are C12 HAA-containing rhamnolipid homologs whose shifted peaks (m/z
  239.162, 385.219) must be guaranteed in the alignment — MMSA fails to capture those
  shifted peaks
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MMSA
  role: Web-based tool that performs the global alignment of all peaks across spectra
    within a molecular family and outputs the consensus peak sets
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Builds the minimum spanning tree over spectra with edges weighted by pairwise
    modified cosine similarity subtracted from 1, then applies topological sorting
- name: GNPS2
  role: Source of molecular networks and MS/MS spectra; users enter a task ID and
    component number or USIs
- name: metabolomics resolver
  role: Resolves Universal Spectrum Identifiers from public sources into spectra that
    MMSA consumes
- name: Python Dash
  role: Web visualization layer that renders the alignment data with clickable peaks,
    set highlighting, m/z range filtering, and SVG export
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:b39cef34e07b8f4d42768c1b00de7bc79e2bc34009ed4654c95ca5adb2d0351d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - Flask
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Plotly
  - Python Dash
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# consensus-peak-set-identification

## Summary

MMSA's consensus-peak-set identification step computes a global multiple alignment of MS/MS spectra within a molecular-network component, producing non-transitively aligned peak sets (m/z + intensity per set) that drive molecular networking formation and expose chemical modifications such as C12 HAA homolog shifts or albicidin modification sites.

## When to use

Use when a molecular family (e.g. ~80 rhamnolipid or albicidin MS/MS spectra from GNPS2) needs a global peak alignment that explains which consensus peaks cause spectra to cluster, rather than only pairwise cosine scores.

## When NOT to use

- Input is a pairwise spectral comparison or a molecular-network edge score only — MMSA is specifically for the alignment of entire molecular families, not pairwise visualization
- The analysis requires exact capture of every small peak alignment (e.g. y2 fragments in albicidin) — the greedy multiple alignment does not fully capture smaller peak sets
- The spectra are C12 HAA-containing rhamnolipid homologs whose shifted peaks (m/z 239.162, 385.219) must be guaranteed in the alignment — MMSA fails to capture those shifted peaks

## Inputs

- Molecular-network component MS/MS spectra from a GNPS2 task (task ID + component number) or FBMN task
- Universal Spectrum Identifiers of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, resolved through the metabolomics resolver
- Preprocessed spectra: top-6 peaks within +/-25 m/z, peaks within +/-17 m/z of precursor excluded, square-root-transformed and normalized intensities

## Outputs

- Consensus peak sets annotated with m/z and intensity values per spectrum in the component
- Alignment JSON saved to ./temp/sets via the 'Process and Save JSON' action
- Interactive Plotly peak-alignment visualization (spectra ordered by precursor m/z) exportable as high-resolution SVG

## How to apply

First preprocess every MS/MS spectrum in the component: keep the top 6 most intense peaks within a +/-25 m/z window, exclude peaks within +/-17 m/z of the precursor ion, apply a square-root transform to intensities, and normalize. Then compute pairwise _cosine_fast modified cosine similarity scores, convert them to edge weights by subtracting each score from 1, and build a minimum spanning tree over the spectra with NetworkX 3.3. Apply topological sorting to fix the node processing order, run the transitive alignment along that order, and finally compute the non-transitively aligned consensus peak sets that constitute the multiple spectral alignment. Via the web interface (GNPS2 Task ID + component number, USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, or FBMN), click 'Process and Save JSON' to write the alignment structure with m/z and intensity values per set to ./temp/sets.

## Related tools

- **MMSA** (Web-based tool that performs the global alignment of all peaks across spectra within a molecular family and outputs the consensus peak sets) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Builds the minimum spanning tree over spectra with edges weighted by pairwise modified cosine similarity subtracted from 1, then applies topological sorting)
- **GNPS2** (Source of molecular networks and MS/MS spectra; users enter a task ID and component number or USIs)
- **metabolomics resolver** (Resolves Universal Spectrum Identifiers from public sources into spectra that MMSA consumes)
- **Python Dash** (Web visualization layer that renders the alignment data with clickable peaks, set highlighting, m/z range filtering, and SVG export)

## Examples

```
python main.py  # then browse to http://localhost:5000, enter GNPS2 task ID + component number, click 'Process and Save JSON'
```

## Evaluation signals

- The alignment JSON exists under ./temp/sets and parses, containing the MST/alignment structure with peak sets annotated with m/z and intensity values for each spectrum in the component
- On the ~80-spectrum rhamnolipid family, the full greedy spanning tree alignment completes in seconds (near real-time), confirming the reported runtime
- For the rhamnolipid family, verify the captured alignment contains the C10 consensus peaks (m/z 135.017, 211.130, 357.189) and note that shifted C12 peaks (m/z 239.162, 385.219) are absent, matching the reported MMSA behavior
- For the albicidin family, the highlighted consensus sets should show b4 and b5 fragments aligned across all derivatives and y5 fragments aligned across most derivatives, with y2 peaks known to be under-captured
- The exported SVG figure reproduces the paper's reported phantom peaks in the C12 and R(OAc) consensus spectra, indicating the greedy alignment artifacts are present as expected

## Limitations

- MMSA did not capture the shifted peaks corresponding to the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219), leaving C12-containing rhamnolipid spectra unaligned with the analogous C10 peak sets
- Smaller peak alignments such as y2 peaks are not fully captured by the greedy multiple alignment
- Phantom peaks can arise in consensus spectra (observed in C12 and R(OAc)) as an artifact of the greedy alignment
- The repository provides no DOI or Zenodo archive for the software, and no hosting URL beyond the local directory name

## Evidence

- [results] first we compute a global alignment of all peaks across spectra within a molecular family: "first we compute a global alignment of all peaks across spectra within a molecular family"
- [results] After transitive alignment, the non-transitively aligned sets were computed: "After transitive alignment, the non-transitively aligned sets were computed"
- [results] The MS/MS is then preprocessed, which includes filtering peaks, intensity transformation, and normalization of peak intensities: "The MS/MS is then preprocessed, which includes filtering peaks, intensity transformation, and normalization of peak intensities"
- [results] peak filtering was applied by selecting the top 6 most intense peaks within a +/− 25 m/z range, excluding peaks close to the precursor ion (within ±17 m/z): "peak filtering was applied by selecting the top 6 most intense peaks within a +/− 25 m/z range, excluding peaks close to the precursor ion (within ±17 m/z)"
- [results] To implement an MST based on the similarity of their peaks, we used NetworkX 3.3: "To implement an MST based on the similarity of their peaks, we used NetworkX 3.3"
- [abstract] MMSA identifies sets of consensus peaks that contribute to alignment of multiple tandem mass spectra: "MMSA identifies sets of consensus peaks that contribute to alignment of multiple tandem mass spectra"
- [readme] git clone <repository-url>
   cd NetworkFamily_MultipleAlignment_Website: "git clone <repository-url>
   cd NetworkFamily_MultipleAlignment_Website"
- [readme] Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`: "Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`"
