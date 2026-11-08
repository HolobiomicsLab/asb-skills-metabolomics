---
name: tanimoto-structural-similarity-filtering
description: Use when multiple top-scoring candidate structures per MS2 feature exist
  from spectral and compound database dereplication and a single prioritized final
  candidate per feature must be selected.
when_to_use_negative:
- 'Ranking or thresholding SIRIUS (CSI:FingerID) candidates alone: the SIRIUS CLI
  does not provide a deterministic probability score, so candidate ranking thresholds
  were discarded in MAW'
- 'During spectral database dereplication: this skill operates on already-derived
  top-scoring candidate structures, not on MS2 spectra matching (cosine similarity)
  itself'
- When the input already contains exactly one candidate per feature and no structural
  deduplication is needed
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Computes pairwise Tanimoto structural similarity between standardized candidate
    SMILES for final candidate selection
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-Py
  role: Python segment of MAW that performs the candidate selection with Tanimoto
    similarity filtering
  repo: https://github.com/zmahnoor14/MAW
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) used to run the MAW-Py candidate-selection
    segment reproducibly
  repo: https://github.com/zmahnoor14/MAW
- name: MetFrag
  role: Candidate annotation source whose MetFrag-style results are pre-filtered with
    a default score threshold of 0.75 before similarity merging
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:4c37d3a3a1ed8dee683192bd87ba76b95ac35605ec98d42737c66c1349ab00fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - MetFrag
  - RDKit
  - maw-py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Tanimoto structural similarity filtering

## Summary

Final-candidate selection step of MAW's Python segment (MAW-Py), where RDKit standardizes top-scoring candidate SMILES from GNPS, HMDB, MassBank, SIRIUS and MetFrag outputs, computes pairwise Tanimoto structural similarity, and merges/discards candidates to select one final structure per feature.

## When to use

Use when multiple top-scoring candidate structures per MS2 feature exist from spectral and compound database dereplication and a single prioritized final candidate per feature must be selected.

## When NOT to use

- Ranking or thresholding SIRIUS (CSI:FingerID) candidates alone: the SIRIUS CLI does not provide a deterministic probability score, so candidate ranking thresholds were discarded in MAW
- During spectral database dereplication: this skill operates on already-derived top-scoring candidate structures, not on MS2 spectra matching (cosine similarity) itself
- When the input already contains exactly one candidate per feature and no structural deduplication is needed

## Inputs

- spectral_results.csv (spectral dereplication results from MAW-R)
- MS1DATA.csv (MS1 feature table)
- GNPS, HMDB and MassBank dereplication result directories
- SIRIUS output files
- MetFrag candidate lists (score-thresholded)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv (one selected candidate per input feature)
- maw_py_log_file.txt (run diagnostics)

## How to apply

Load the spectral dereplication results (spectral_results.csv) and the MS1 feature table (MS1DATA.csv) from the upstream MAW-R run, then parse the GNPS, HMDB and MassBank dereplication directories plus SIRIUS output, keeping top-scoring candidate SMILES per feature (MetFrag candidates are pre-filtered with a default --score_thresh of 0.75). Standardize the candidate structures with RDKit and compute pairwise Tanimoto structural similarity, discarding candidates with similarity below 0.85 and merging candidates with similarity >=0.99 under the identity rule. Rank the surviving candidates by the source prioritization scheme — candidates supported by three sources first, then two sources, then GNPS > SIRIUS > MassBank/HMDB — and select exactly one final candidate per feature, writing the merged results table. The rationale is that MAW prioritizes structures that recur among the top candidates of most annotation sources, using Tanimoto similarity of >=0.99 for further candidate-selection calculations.

## Related tools

- **RDKit** (Computes pairwise Tanimoto structural similarity between standardized candidate SMILES for final candidate selection) — https://github.com/zmahnoor14/MAW
- **MAW-Py** (Python segment of MAW that performs the candidate selection with Tanimoto similarity filtering) — https://github.com/zmahnoor14/MAW
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) used to run the MAW-Py candidate-selection segment reproducibly) — https://github.com/zmahnoor14/MAW
- **MetFrag** (Candidate annotation source whose MetFrag-style results are pre-filtered with a default score threshold of 0.75 before similarity merging) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64
```

## Evaluation signals

- The output file file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one selected candidate per input feature
- No surviving candidate pair has unexamined Tanimoto similarity: candidates with similarity >=0.99 are merged under the identity rule
- All candidates below the 0.85 Tanimoto similarity cutoff have been discarded before ranking
- MetFrag-style candidates were pre-filtered with the default score threshold of 0.75
- Run diagnostics are captured in maw_py_log_file.txt

## Limitations

- Large databases such as PubChem can lead to a larger list of candidates with the actual structure ranked low (e.g. top 100th candidate), limiting selection quality
- SIRIUS CLI does not provide a deterministic probability score, so candidate ranking thresholds for its output were discarded as case-dependent
- A mismatch between spectra submitted to spectral databases and structures in compound databases leads to a low number of annotated features entering this step

## Evidence

- [abstract] The final candidate selection is performed using the cheminformatics tool RDKit in the Python segment (MAW-Py): "The final candidate selection is performed using the cheminformatics tool RDKit in the Python segment (MAW-Py)"
- [discussion] We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [methods] '--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)': "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [methods] docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64: "docker pull zmahnoor/maw-py:1.0.7 --platform=linux/amd64"
- [intro] It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature.: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
