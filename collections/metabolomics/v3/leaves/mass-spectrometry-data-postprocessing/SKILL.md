---
name: mass-spectrometry-data-postprocessing
description: Use when mzML MS2 features already have raw spectral matches and SIRIUS/MetFrag
  candidate lists and you must merge them, apply score thresholds, and prioritize
  one annotated candidate per feature.
when_to_use_negative:
- Input is raw .mzML MS2 data that has not yet been through MAW-R spectral database
  dereplication and SIRIUS/MetFrag in silico annotation — run the upstream MAW-R segment
  first
- You need deterministic probability-based ranking thresholds for SIRIUS candidates
  — SIRIUS CLI does not provide a deterministic probability score, so thresholds were
  discarded in MAW
- The goal is annotation against very large compound databases like PubChem, where
  the correct structure can be pushed to low ranks (e.g. top 100th candidate)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MAW-Py
  role: Python segment hosting the post-processing scripts that merge spectral and
    in silico results and select the final candidate per feature
  repo: https://github.com/zmahnoor14/MAW
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) in which the MAW-Py post-processing scripts
    are executed for reproducibility
  repo: https://github.com/zmahnoor14/MAW
- name: RDKit
  role: Cheminformatics tool used in the final candidate selection step for structural/Tanimoto
    similarity matching
  repo: https://www.rdkit.org/
- name: Spectra
  role: R package whose spectral database dereplication output (spectral_results.csv)
    is merged with in silico results
- name: SIRIUS
  role: In silico annotation tool (CSI:FingerID) whose candidate lists feed the merged-results
    post-processing
- name: MetFrag
  role: In silico candidate generator whose results are filtered with the 0.75 score
    threshold during post-processing
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:4de71fe11cb10351ae387784b695fdea0b6146342805c32b2522e33eefb32753
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - MetFrag
  - RDKit
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# mass-spectrometry-data-postprocessing

## Summary

Merge spectral database dereplication results (GNPS/HMDB/MassBank via Spectra) with in silico annotation results (SIRIUS or MetFrag) and select one final candidate per MS2 feature, using MAW-Py post-processing scripts with RDKit and a MetFrag score threshold of 0.75.

## When to use

Use when mzML MS2 features already have raw spectral matches and SIRIUS/MetFrag candidate lists and you must merge them, apply score thresholds, and prioritize one annotated candidate per feature.

## When NOT to use

- Input is raw .mzML MS2 data that has not yet been through MAW-R spectral database dereplication and SIRIUS/MetFrag in silico annotation — run the upstream MAW-R segment first
- You need deterministic probability-based ranking thresholds for SIRIUS candidates — SIRIUS CLI does not provide a deterministic probability score, so thresholds were discarded in MAW
- The goal is annotation against very large compound databases like PubChem, where the correct structure can be pushed to low ranks (e.g. top 100th candidate)

## Inputs

- spectral_results.csv from Spectra dereplication against GNPS/HMDB/MassBank
- SIRIUS or MetFrag in silico results (from .ms files processed with run_sirius, or MetFrag CLI output)
- GNPS/HMDB/MassBank results directories
- MS1DATA.csv (MS1 feature table)
- MetFrag score threshold (--score_thresh, default 0.75)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv with one final candidate per feature
- per-feature structural, formula, and chemical class annotations for coverage tallying

## How to apply

Run the MAW-Py post-processing script (Workflow_Python_Script_all_MetFrag.py or Workflow_Python_Script_all_SIRIUS.py) inside the zmahnoor/maw-py:1.0.7 container, supplying spectral_results.csv, the GNPS/HMDB/MassBank results directories, and MS1DATA.csv as inputs. Apply a MetFrag score threshold of 0.75 (--score_thresh default) so only high-scoring in silico candidates are considered; the scripts combine these with Spectra cosine-similarity spectral matches (with MAW-defined metrics required to reach at least 0.5 in addition to high cosine similarity) and use RDKit to compute Tanimoto similarity for structural matching. The script merges spectral and in silico results and selects a single final candidate per feature, writing file_id_mergedResults-with-one-Candidates.csv. Verify correct application by checking that exactly one candidate per feature is retained and that annotation coverage tallies (e.g. MAW 100% structural, 97.3% chemical class, 100% formula on the 37-feature standards dataset) match the reported comparison values.

## Related tools

- **MAW-Py** (Python segment hosting the post-processing scripts that merge spectral and in silico results and select the final candidate per feature) — https://github.com/zmahnoor14/MAW
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) in which the MAW-Py post-processing scripts are executed for reproducibility) — https://github.com/zmahnoor14/MAW
- **RDKit** (Cheminformatics tool used in the final candidate selection step for structural/Tanimoto similarity matching) — https://www.rdkit.org/
- **Spectra** (R package whose spectral database dereplication output (spectral_results.csv) is merged with in silico results)
- **SIRIUS** (In silico annotation tool (CSI:FingerID) whose candidate lists feed the merged-results post-processing)
- **MetFrag** (In silico candidate generator whose results are filtered with the 0.75 score threshold during post-processing) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
python Workflow_Python_Script_all_MetFrag.py --spectral_results spectral_results.csv --metfrag_results MetFragResults/ --ms1_data MS1DATA.csv --score_thresh 0.75
```

## Evaluation signals

- file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one final candidate per feature
- Recomputed annotation coverage matches Table 4 values: MAW 100% structural, 97.3% chemical class, 100% formula; Spectra 75.7%; SIRIUS 70.2%; MetFrag+PubChem 32.4% (12/37) on the standards dataset
- Only MetFrag candidates scoring at or above the 0.75 --score_thresh survive filtering
- MAW-defined metrics for spectral matches reach at least 0.5 in addition to a high cosine similarity score
- All candidate structures carry valid Identifier, InChI, SMILES, and molecular_weight fields consistent with the structure database CSV format

## Limitations

- MAW-defined 0.5 threshold may miss candidates with low cosine similarity scores as false negatives
- Only SIRIUS (CSI:FingerID) is integrated for compound databases in the first MAW version; MetFrag with PubChemLite/COCONUT is planned future work
- Large databases such as PubChem can push the actual structure to a low rank (e.g. top 100th candidate)
- Public spectral databases are not fully standardized, which can cause misannotation (e.g. betaine annotated as reduced-state trimethylglycine)
- Mismatch between experimental spectra in spectral databases and structures in compound databases limits how many features can be annotated

## Evidence

- [intro] MAW performs MS2 data preprocessing, spectral database dereplication, compound database dereplication, candidate selection, and chemical compound classification: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification using various"
- [abstract] Final candidate selection performed using RDKit in MAW-Py: "The final candidate selection is performed using the cheminformatics tool RDKit in the Python segment (MAW-Py)"
- [methods] Score threshold for MetFrag results is 0.75: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] MAW-defined metrics require at least 0.5 matching score plus high cosine similarity: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [readme] README states workflow combines Spectra dereplication, SIRIUS/MetFrag, and RDKit-based final selection: "It performs spectral database dereplication using R Package [Spectra](https://rformassspectrometry.github.io/Spectra/) and compound database dereplication using"
