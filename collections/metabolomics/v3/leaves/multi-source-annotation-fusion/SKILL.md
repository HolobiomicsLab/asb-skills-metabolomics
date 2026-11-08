---
name: multi-source-annotation-fusion
description: Use when you have per-feature candidate lists from several spectral/compound
  annotation sources and must reduce them to a single, source-supported chemical structure
  (SMILES) per feature for downstream network or reporting steps.
when_to_use_negative:
- Input is raw MS2 mzML data still needing preprocessing and spectral dereplication
  — that belongs to the upstream MAW-R segment (Spectra, SIRIUS), not final candidate
  selection.
- Only one annotation source is available, so cross-source candidate fusion and source-priority
  ranking provide no added discrimination.
- SIRIUS CLI output with no deterministic probability score is the only compound annotation
  and you need score-based ranking thresholds — the article notes such thresholds
  were discarded as case-dependent.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Standardizes candidate structures and computes Tanimoto structural similarity
    used to merge and prioritize candidates
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python workflow segment in which the final candidate selection is implemented
  repo: https://github.com/zmahnoor14/MAW
- name: PubChemPy
  role: Used alongside RDKit in the Python final-selection segment of MAW
  repo: https://pubchempy.readthedocs.io/en/latest/
- name: Spectra
  role: Upstream R package producing spectral dereplication results that feed candidate
    fusion
  repo: https://rformassspectrometry.github.io/Spectra/
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) delivering the reproducible MAW-Py candidate
    selection step
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:cbb72c58387b669c2afca167b481bd80e821adc7e5ee6d889683ce6a0cb3abfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-Py
  - PubChemPy
  - RDKit
  - Spectra
  - maw-py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multi-source-annotation-fusion

## Summary

Final-stage candidate fusion in the MAW workflow: merge and rank top-scoring annotation candidates from multiple dereplication sources (GNPS, HMDB, MassBank, SIRIUS/MetFrag) using RDKit structural similarity, and select exactly one final candidate chemical structure per MS2 feature.

## When to use

Use when you have per-feature candidate lists from several spectral/compound annotation sources and must reduce them to a single, source-supported chemical structure (SMILES) per feature for downstream network or reporting steps.

## When NOT to use

- Input is raw MS2 mzML data still needing preprocessing and spectral dereplication — that belongs to the upstream MAW-R segment (Spectra, SIRIUS), not final candidate selection.
- Only one annotation source is available, so cross-source candidate fusion and source-priority ranking provide no added discrimination.
- SIRIUS CLI output with no deterministic probability score is the only compound annotation and you need score-based ranking thresholds — the article notes such thresholds were discarded as case-dependent.

## Inputs

- spectral dereplication results table (spectral_results.csv) from MAW-R
- MS1 feature table (MS1DATA.csv)
- GNPS, HMDB and MassBank dereplication result directories
- SIRIUS (CSI:FingerID) output candidates
- MetFrag candidate lists (filtered at score threshold 0.75)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv (one selected candidate SMILES per feature)
- maw_py_log_file.txt (run diagnostics)
- per-feature chemical structures importable to a chemical structure similarity network

## How to apply

Load the spectral dereplication results (spectral_results.csv) and the MS1 feature table (MS1DATA.csv) from the upstream MAW-R run, then parse the GNPS, HMDB and MassBank dereplication directories plus SIRIUS (or MetFrag, with candidates pre-filtered at --score_thresh 0.75) output, keeping the top-scoring candidate SMILES per feature. Standardize candidate structures with RDKit and compute pairwise Tanimoto structural similarity, merging candidates whose similarity is >=0.99 under the identity rule and discarding those below the lower similarity cutoff. Rank survivors by source support: candidates backed by three annotation sources first, then two, then by source priority GNPS > SIRIUS > MassBank/HMDB. Write exactly one final candidate per feature to file_id_mergedResults-with-one-Candidates.csv and capture run diagnostics in maw_py_log_file.txt.

## Related tools

- **RDKit** (Standardizes candidate structures and computes Tanimoto structural similarity used to merge and prioritize candidates) — https://www.rdkit.org/
- **MAW-Py** (Python workflow segment in which the final candidate selection is implemented) — https://github.com/zmahnoor14/MAW
- **PubChemPy** (Used alongside RDKit in the Python final-selection segment of MAW) — https://pubchempy.readthedocs.io/en/latest/
- **Spectra** (Upstream R package producing spectral dereplication results that feed candidate fusion) — https://rformassspectrometry.github.io/Spectra/
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) delivering the reproducible MAW-Py candidate selection step) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64
```

## Evaluation signals

- file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one selected candidate per input feature
- Candidates with pairwise Tanimoto similarity >=0.99 are merged as identical; only candidates passing the similarity rule survive ranking
- Final candidate ranking respects the source prioritization: 3-source candidates first, then 2-source, then GNPS > SIRIUS > MassBank/HMDB
- Run diagnostics are present in maw_py_log_file.txt with no per-feature selection errors
- MetFrag-sourced candidates entering fusion were pre-filtered at the default score threshold of 0.75

## Limitations

- Larger databases such as PubChem can produce large candidate lists with the actual structure ranked as low as the top 100th candidate, so fusion is only as good as the top-scoring inputs.
- The MAW-defined matching metrics with a 0.5 threshold plus high cosine similarity reduce false positives but can miss candidates with low cosine similarity as false negatives.
- Public spectral databases are not fully standardized, which can propagate misannotations (e.g. betaine annotated as reduced-state trimethylglycine) into the fused candidate pool.

## Evidence

- [abstract] The final candidate selection is performed using the cheminformatics tool RDKit: "The final candidate selection is performed using the cheminformatics tool RDKit"
- [discussion] Tanimoto for structural matching: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [methods] MetFrag score threshold: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [readme] README: selection in Python: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)."
- [intro] Network and final candidate per feature: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
