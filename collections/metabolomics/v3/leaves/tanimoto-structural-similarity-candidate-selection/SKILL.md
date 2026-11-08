---
name: tanimoto-structural-similarity-candidate-selection
description: Use when prioritizing a single final candidate per feature from MetFrag
  (e.g. COCONUT-backed) or SIRIUS annotation results for untargeted MS2 .mzML data,
  by combining cosine-similarity dereplication, MAW-defined metrics, MetFrag scores
  above 0.75, and RDKit Tanimoto structural matching.
when_to_use_negative:
- When the feature already has a confident spectral database match (e.g. HMDB/MassBank
  cosine hit), since large compound-database dereplication plus Tanimoto rescoring
  can push the actual structure to low ranks (e.g. top 100th candidate)
- When relying on SIRIUS probability-like thresholds for candidate pruning — SIRIUS
  CLI provides no deterministic probability score, so ranking thresholds were discarded
  in MAW
- When annotating complex macrocyclic structures (e.g. Marchantins) without a suspect
  list, where Tanimoto ranking may not overcome difficult fragmentation patterns
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Computes Tanimoto similarity for structural matching during final candidate
    selection in MAW-Py
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-Py
  role: Python workflow segment (maw-py container) that postprocesses MetFrag results
    and performs final candidate selection
  repo: https://github.com/zmahnoor14/MAW
- name: MetFrag
  role: Supplies the in silico candidate lists (e.g. against COCONUT) that are filtered
    with --score_thresh 0.75 before Tanimoto-based selection
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: Spectra
  role: Provides cosine-similarity spectral matching scores combined with the Tanimoto
    structural score in candidate ranking
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:5cc833ed7791b60342356cbb127779b8eca7029286a48aa829ce8ae7e758d5b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - MetFrag
  - RDKit
  - Spectra
  - maw-py
  - maw-r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tanimoto-structural-similarity-candidate-selection

## Summary

MAW's final candidate selection stage, in which RDKit computes Tanimoto structural similarity in the MAW-Py Python segment to prioritize one final candidate per MS2 feature from MetFrag/SIRIUS candidate lists that already passed spectral-database dereplication and the MetFrag score threshold.

## When to use

Use when prioritizing a single final candidate per feature from MetFrag (e.g. COCONUT-backed) or SIRIUS annotation results for untargeted MS2 .mzML data, by combining cosine-similarity dereplication, MAW-defined metrics, MetFrag scores above 0.75, and RDKit Tanimoto structural matching.

## When NOT to use

- When the feature already has a confident spectral database match (e.g. HMDB/MassBank cosine hit), since large compound-database dereplication plus Tanimoto rescoring can push the actual structure to low ranks (e.g. top 100th candidate)
- When relying on SIRIUS probability-like thresholds for candidate pruning — SIRIUS CLI provides no deterministic probability score, so ranking thresholds were discarded in MAW
- When annotating complex macrocyclic structures (e.g. Marchantins) without a suspect list, where Tanimoto ranking may not overcome difficult fragmentation patterns

## Inputs

- spectral_results.csv from MAW-R spectral database dereplication
- GNPS, HMDB, and MassBank dereplication result directories
- MetFrag candidate results (e.g. /insilico/MetFrag/coconut) from compound database dereplication
- MS1DATA.csv (MS1 feature table)
- Candidate SMILES/structures from compound databases such as COCONUT_Jan2022.csv

## Outputs

- file_id_mergedResults-with-one-Candidates.csv — merged, filtered candidate list with one prioritized candidate per MS2 feature
- Tanimoto-similarity-ranked candidate selections assignable to a chemical structure similarity network
- maw_py_log_file.txt run log

## How to apply

Run MAW-Py (Workflow_Python_Script_all_MetFrag.py in the maw-py container) on the spectral_results.csv produced by MAW-R, the GNPS/HMDB/MassBank dereplication directories, MS1DATA.csv, and the MetFrag results (e.g. under /insilico/MetFrag/coconut). Apply the MetFrag score threshold (default 0.75 via --score_thresh) to filter candidates, then require MAW-defined metrics of at least 0.5 matching score in addition to high cosine similarity from the Spectra-based spectral dereplication. Use RDKit's Tanimoto similarity score for the structural matching step to rank and select the final candidate per feature, yielding file_id_mergedResults-with-one-Candidates.csv with a prioritized candidate list per feature. Note that no threshold is applied to SIRIUS outputs because the SIRIUS CLI does not provide a deterministic probability score, so its ranking contribution is used as-is alongside Spectra (CSI:FingerID) scoring.

## Related tools

- **RDKit** (Computes Tanimoto similarity for structural matching during final candidate selection in MAW-Py) — https://github.com/zmahnoor14/MAW
- **MAW-Py** (Python workflow segment (maw-py container) that postprocesses MetFrag results and performs final candidate selection) — https://github.com/zmahnoor14/MAW
- **MetFrag** (Supplies the in silico candidate lists (e.g. against COCONUT) that are filtered with --score_thresh 0.75 before Tanimoto-based selection) — https://github.com/ipb-halle/MetFragRelaunched
- **Spectra** (Provides cosine-similarity spectral matching scores combined with the Tanimoto structural score in candidate ranking) — https://github.com/zmahnoor14/MAW

## Examples

```
python Workflow_Python_Script_all_MetFrag.py --score_thresh 0.75  # run in zmahnoor/maw-py on spectral_results.csv + GNPS/HMDB/MassBank dirs + MS1DATA.csv
```

## Evaluation signals

- All retained MetFrag candidates have scores above the --score_thresh value (default 0.75)
- MAW-defined metrics meet at least a 0.5 matching score in addition to high cosine similarity
- file_id_mergedResults-with-one-Candidates.csv contains exactly one prioritized candidate per MS2 feature
- For validation cases like Marchantia polymorpha, known compounds (Perrottetin E rank 10, Marchantin H rank 10, Marchantin M rank 2, Marchantin D rank 2, Marchantin F rank 8) appear within the top 25 candidates
- maw_py_log_file.txt records the run without errors and candidates carry RDKit-computable structures

## Limitations

- Larger compound databases such as PubChem can lead to larger candidate lists with the actual structure pushed to lower ranks (e.g. top 100th candidate), degrading Tanimoto-based top-N selection
- Macrocyclic structures like Marchantins are hard to annotate because of complex fragmentation patterns; suspect lists are recommended
- Mismatch between experimental spectra in spectral databases and structures in compound databases leads to few annotated features, limiting what candidate selection can rescue
- SIRIUS CLI outputs no deterministic probability score, so threshold-based filtering of its candidates is not possible

## Evidence

- [discussion] Tanimoto scoring rationale: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [methods] MetFrag score threshold parameter: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] MAW-defined metric threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [readme] RDKit in MAW-Py selection: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)"
- [abstract] Final selection abstract statement: "The final candidate selection is performed using the cheminformatics tool RDKit"
