---
name: metabolite-candidate-prioritization
description: Use when multiple annotation tools or spectral databases (GNPS, SIRIUS,
  MassBank, HMDB, MetFrag) have each produced per-feature candidate lists for an untargeted
  MS2 dataset, and you must merge them into one prioritized structure per feature.
when_to_use_negative:
- The candidate lists from individual annotation tools/databases have not yet been
  generated (this skill consumes their outputs, not raw mzML spectra)
- The goal is per-source ranking or network visualization of all candidates rather
  than collapsing to a single final structure per feature
- A deterministic per-tool probability threshold is required for SIRIUS candidates
  — the SIRIUS CLI provides no deterministic probability score, so thresholds were
  discarded by the MAW authors
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MAW-Py
  role: Python segment of MAW implementing the final candidate selection logic
  repo: https://github.com/zmahnoor14/MAW
- name: RDKit
  role: Structure standardization, Tanimoto similarity computation, and final candidate
    selection
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) packaging the candidate-selection code
- name: SIRIUS
  role: Upstream in silico annotation source whose candidates feed the prioritization
- name: MetFrag
  role: Optional compound database annotation tool whose candidates enter selection
    with a 0.75 score threshold
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:d0f6a7345b9ddaa9524c3dd9cc1502afd46b5e560b24c219be94c8b4d103d3bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - MetFrag
  - RDKit
  - SIRIUS
  - maw-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-candidate-prioritization

## Summary

The MAW-Py stage that automates selection of one final candidate structure per MS1 feature from top-scoring candidates of multiple annotation sources (GNPS, SIRIUS/CSI:FingerID, MassBank, HMDB, MetFrag) using RDKit-based structural standardization, Tanimoto similarity merging, and a source-priority ranking scheme.

## When to use

Use when multiple annotation tools or spectral databases (GNPS, SIRIUS, MassBank, HMDB, MetFrag) have each produced per-feature candidate lists for an untargeted MS2 dataset, and you must merge them into one prioritized structure per feature.

## When NOT to use

- The candidate lists from individual annotation tools/databases have not yet been generated (this skill consumes their outputs, not raw mzML spectra)
- The goal is per-source ranking or network visualization of all candidates rather than collapsing to a single final structure per feature
- A deterministic per-tool probability threshold is required for SIRIUS candidates — the SIRIUS CLI provides no deterministic probability score, so thresholds were discarded by the MAW authors

## Inputs

- spectral dereplication results (spectral_results.csv) from MAW-R/Spectra
- MS1 feature table (MS1DATA.csv)
- GNPS, HMDB and MassBank dereplication result directories
- SIRIUS (CSI:FingerID) output with candidate SMILES
- MetFrag candidate results (filtered at score_thresh = 0.75)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv (exactly one selected candidate per feature)
- maw_py_log_file.txt (run diagnostics)

## How to apply

Load the spectral dereplication results (spectral_results.csv) and MS1 feature table (MS1DATA.csv) from the upstream MAW-R run, then parse the GNPS, HMDB and MassBank dereplication directories plus SIRIUS output, keeping top-scoring candidate SMILES per feature (MetFrag candidates are pre-filtered with --score_thresh 0.75). Standardize the candidate structures with RDKit and compute pairwise Tanimoto structural similarity, merging candidates with similarity >=0.99 under the identity rule and discarding those below 0.85. Rank survivors by source-support prioritization: candidates backed by three sources first, then two, with source precedence GNPS > SIRIUS > MassBank/HMDB. Select one final candidate per feature and write the merged results table; a deterministic SIRIUS probability threshold was deliberately avoided because the SIRIUS CLI provides no deterministic probability score.

## Related tools

- **MAW-Py** (Python segment of MAW implementing the final candidate selection logic) — https://github.com/zmahnoor14/MAW
- **RDKit** (Structure standardization, Tanimoto similarity computation, and final candidate selection)
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) packaging the candidate-selection code)
- **SIRIUS** (Upstream in silico annotation source whose candidates feed the prioritization)
- **MetFrag** (Optional compound database annotation tool whose candidates enter selection with a 0.75 score threshold) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64
```

## Evaluation signals

- file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one selected candidate per input MS1 feature
- Candidates merged under the identity rule have RDKit Tanimoto similarity >= 0.99; no retained distinct candidate falls below the 0.85 similarity cutoff
- Ranking respects the source-priority order: three-source support first, then two-source, then GNPS > SIRIUS > MassBank/HMDB
- MetFrag-derived candidates admitted into selection all meet the default score threshold of 0.75
- Run diagnostics are captured in maw_py_log_file.txt with no unresolved features

## Limitations

- Only SIRIUS (CSI:FingerID) was integrated for compound database annotation in the first MAW version; MetFrag is not yet fully integrated
- Large compound databases such as PubChem can push the actual structure to low ranks (e.g. top 100th candidate), degrading prioritization
- SIRIUS CLI lacks a deterministic probability score, so ranking thresholds for its candidates would differ case by case and were discarded
- Huge mismatch between spectra in spectral databases and structures in compound databases leads to a low number of annotatable features

## Evidence

- [abstract] Final candidate selection is performed using the cheminformatics tool RDKit: "The final candidate selection is performed using the cheminformatics tool RDKit"
- [intro] MAW prioritizes a final candidate per feature: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
- [discussion] Tanimoto similarity used for structural matching: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [methods] MetFrag score threshold parameter: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [readme] README describes final selection in Python: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)."
