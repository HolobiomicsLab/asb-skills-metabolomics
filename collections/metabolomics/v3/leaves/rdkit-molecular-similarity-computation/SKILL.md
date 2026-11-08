---
name: rdkit-molecular-similarity-computation
description: Use when a metabolomics annotation workflow must consolidate multiple
  top-scoring candidate structures per MS2 feature into a single final candidate using
  structural similarity comparisons rather than spectral scores alone.
when_to_use_negative:
- Input is a raw MS2 mzML file or an unprocessed feature list — spectral dereplication
  in MAW-R must run first
- You need to annotate spectra against spectral databases — this skill only performs
  final structural selection, not cosine-similarity dereplication via Spectra
- SIRIUS probability-score-based ranking is desired — the article notes the SIRIUS
  CLI does not provide a deterministic probability score, so thresholds were discarded
tools:
- name: RDKit
  role: Standardizes candidate structures and computes pairwise Tanimoto structural
    similarity for final candidate selection
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python workflow segment that embeds the RDKit-based candidate selection and
    writes the merged results table
  repo: https://github.com/zmahnoor14/MAW
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) packaging the MAW-Py candidate-selection
    step
  repo: https://github.com/zmahnoor14/MAW
- name: PubChemPy
  role: Used alongside RDKit in MAW-Py for final candidate selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:ee4e72fcefd5d43b6405e51a8d391c5591025f7743cdaf73f7ef8b8a5dd0960a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-Py
  - PubChemPy
  - RDKit
  - maw-py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rdkit-molecular-similarity-computation

## Summary

Computation of pairwise structural similarity (Tanimoto score) between candidate metabolite structures using RDKit, as the basis for MAW's final candidate merging and prioritization in the Python segment (MAW-Py).

## When to use

Use when a metabolomics annotation workflow must consolidate multiple top-scoring candidate structures per MS2 feature into a single final candidate using structural similarity comparisons rather than spectral scores alone.

## When NOT to use

- Input is a raw MS2 mzML file or an unprocessed feature list — spectral dereplication in MAW-R must run first
- You need to annotate spectra against spectral databases — this skill only performs final structural selection, not cosine-similarity dereplication via Spectra
- SIRIUS probability-score-based ranking is desired — the article notes the SIRIUS CLI does not provide a deterministic probability score, so thresholds were discarded

## Inputs

- spectral dereplication results table (spectral_results.csv) from MAW-R
- MS1 feature table (MS1DATA.csv)
- GNPS, HMDB and MassBank dereplication directories with candidate SMILES
- SIRIUS output directory (CSI:FingerID candidates)
- MetFrag candidate lists (score threshold 0.75)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv (one selected candidate per input feature)
- merged results table of standardized candidate structures with Tanimoto similarity values
- run diagnostics in maw_py_log_file.txt

## How to apply

Load the top-scoring candidate SMILES per feature from upstream dereplication outputs (GNPS, HMDB, MassBank, SIRIUS; MetFrag-style candidates pre-filtered with --score_thresh default 0.75). Standardize the structures and compute pairwise Tanimoto similarity in RDKit: merge candidates under the identity rule when similarity is >=0.99 and discard distant candidates (similarity below 0.85) before ranking. Rank survivors by source support (candidates backed by three annotation sources first, then two, then GNPS > SIRIUS > MassBank/HMDB) and select exactly one final candidate per feature. The rationale is that MAW prioritizes structures corroborated across annotation sources, improving candidate ranking over any single tool. Diagnostics should be captured in maw_py_log_file.txt and the merged table written as file_id_mergedResults-with-one-Candidates.csv.

## Related tools

- **RDKit** (Standardizes candidate structures and computes pairwise Tanimoto structural similarity for final candidate selection) — https://www.rdkit.org/
- **MAW-Py** (Python workflow segment that embeds the RDKit-based candidate selection and writes the merged results table) — https://github.com/zmahnoor14/MAW
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) packaging the MAW-Py candidate-selection step) — https://github.com/zmahnoor14/MAW
- **PubChemPy** (Used alongside RDKit in MAW-Py for final candidate selection)

## Examples

```
docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64
```

## Evaluation signals

- Output file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one selected candidate per input MS1 feature
- Candidates merged under the identity rule have Tanimoto similarity >=0.99; no retained pair falls below the 0.85 similarity cutoff
- Selected candidates are consistent with the source-prioritization order: 3-source candidates > 2-source > GNPS > SIRIUS > MassBank/HMDB
- MetFrag inputs were pre-filtered at --score_thresh 0.75 before similarity computation
- Run diagnostics recorded in maw_py_log_file.txt with no unhandled parse errors from the GNPS/HMDB/MassBank directories

## Limitations

- The MAW-defined matching threshold (>=0.5 plus high cosine similarity upstream) may miss candidates with low cosine similarity as false negatives
- SIRIUS CLI provides no deterministic probability score, so ranking thresholds for its candidates were discarded
- Public spectral databases are not fully standardized (e.g. betaine annotated as reduced-state trimethylglycine), which can propagate misannotations into the candidate pool
- Large databases such as PubChem can push the actual structure to a low rank (e.g. top 100th candidate), limiting what similarity-based merging can recover

## Evidence

- [abstract] Final candidate selection via RDKit in MAW-Py: "The final candidate selection is performed using the cheminformatics tool RDKit"
- [discussion] Tanimoto similarity for structural matching: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [readme] Final selection in Python with RDKit and PubChemPy: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)."
- [intro] Per-feature prioritization of a final candidate: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
- [methods] MetFrag candidate score threshold: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
