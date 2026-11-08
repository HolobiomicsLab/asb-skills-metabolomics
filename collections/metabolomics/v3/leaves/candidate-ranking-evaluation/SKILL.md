---
name: candidate-ranking-evaluation
description: Use when a workflow must prioritize and validate one annotation candidate
  per MS2 feature — e.g. merging spectral database dereplication (GNPS/HMDB/MassBank)
  with MetFrag/SIRIUS candidate lists and checking ranks, scores, or coverage percentages.
when_to_use_negative:
- Features have no plausible spectral or compound database matches — the paper notes
  a 'huge mismatch between submitted experimental spectra to spectral databases and
  chemical structures present in compound databases', leading to few annotatable features
- Target structures are macrocyclic (e.g. Marchantin) with complex fragmentation patterns
  — suspect lists are recommended instead of relying on top-ranked in silico candidates
- SIRIUS-only pipelines where a deterministic probability score is needed — SIRIUS
  CLI provides none, so ranking thresholds were explicitly discarded by the authors
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Spectra
  role: R package for spectral database dereplication (GNPS/HMDB/MassBank) with cosine
    similarity scoring
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MetFrag
  role: In silico candidate generation; results filtered with a 0.75 score threshold
    during candidate selection
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: SIRIUS
  role: In silico annotation (CSI:FingerID) providing candidate rankings used in merged
    candidate selection
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: RDKit
  role: Cheminformatics tool used in MAW-Py for final candidate selection via Tanimoto
    structural similarity
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python post-processing segment that merges spectral and in silico results
    and performs final candidate selection (--score_thresh 0.75)
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-R
  role: R segment performing MS2 preprocessing and spectral/compound database dereplication
    that feeds candidate ranking
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:e55fedaaff0f38d1896f2c5b9ccda2f395d7a9ab96b1226dad68bb86036d2a7d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-Py
  - MAW-R
  - MetFrag
  - RDKit
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# candidate-ranking-evaluation

## Summary

Evaluates and prioritizes metabolite annotation candidates from untargeted MS2 data by merging spectral database dereplication (GNPS/HMDB/MassBank cosine similarity) with in silico candidate lists (MetFrag, SIRIUS), then selecting one final candidate per feature and validating ranks/coverage against reported benchmarks.

## When to use

Use when a workflow must prioritize and validate one annotation candidate per MS2 feature — e.g. merging spectral database dereplication (GNPS/HMDB/MassBank) with MetFrag/SIRIUS candidate lists and checking ranks, scores, or coverage percentages.

## When NOT to use

- Features have no plausible spectral or compound database matches — the paper notes a 'huge mismatch between submitted experimental spectra to spectral databases and chemical structures present in compound databases', leading to few annotatable features
- Target structures are macrocyclic (e.g. Marchantin) with complex fragmentation patterns — suspect lists are recommended instead of relying on top-ranked in silico candidates
- SIRIUS-only pipelines where a deterministic probability score is needed — SIRIUS CLI provides none, so ranking thresholds were explicitly discarded by the authors

## Inputs

- MS2 .mzML data files
- Spectral database .rda files (gnps.rda, hmdb.rda, mbankNIST.rda)
- Compound database CSV (e.g. COCONUT_Jan2022.csv or local structure DB with Identifier, InChI, SMILES, molecular_weight columns)
- MetFrag parameter file metparam_list.txt and MetFragCommandLine-2.5.0.jar
- MS1DATA.csv and spectral_results.csv

## Outputs

- file_id_mergedResults-with-one-Candidates.csv with one final candidate per feature
- Per-feature candidate ranks and scores (MetFrag score, cosine similarity, Tanimoto similarity)
- Annotation coverage percentages comparable to Table 4 (MAW structural/class/formula, Spectra 75.7%, SIRIUS 70.2%, MetFrag 32.4%)

## How to apply

Run MAW-R inside the maw-r container on .mzML files with a 15 ppm precursor tolerance for spectral database dereplication against GNPS/HMDB/MassBank, generating candidate lists from MetFrag (against a COCONUT or local structure CSV with Identifier, InChI, SMILES, molecular_weight columns) or SIRIUS 5 (ppm_max 5, ppm_max_ms2 15, 30 candidates, profile 'orbitrap', db 'coconut'). Post-process in MAW-Py with a MetFrag score threshold of 0.75 (--score_thresh) to merge spectral_results.csv, GNPS/HMDB/MassBank dereplication directories, and MS1DATA.csv into one final candidate per feature. Candidate filtering combines a high cosine similarity score with MAW-defined metrics that should each reach at least 0.5, and the Tanimoto similarity score for structural matching, computed via RDKit in the final selection step. Evaluate the ranking by recomputing annotation coverage percentages (Table 4: MAW structural 100%, Spectra matches 75.7%, SIRIUS 70.2%, MetFrag CLI 2.5.0 with PubChem 32.4% on the 37-feature standards dataset) or by checking that target compounds (e.g. Marchantin D/M/F/H and Perrottetin E) appear within the top 25 candidates at expected ranks (2, 2, 8, 10, 10).

## Related tools

- **Spectra** (R package for spectral database dereplication (GNPS/HMDB/MassBank) with cosine similarity scoring) — https://rformassspectrometry.github.io/Spectra/
- **MetFrag** (In silico candidate generation; results filtered with a 0.75 score threshold during candidate selection) — https://github.com/ipb-halle/MetFragRelaunched
- **SIRIUS** (In silico annotation (CSI:FingerID) providing candidate rankings used in merged candidate selection) — https://bio.informatik.uni-jena.de/software/sirius/
- **RDKit** (Cheminformatics tool used in MAW-Py for final candidate selection via Tanimoto structural similarity) — https://www.rdkit.org/
- **MAW-Py** (Python post-processing segment that merges spectral and in silico results and performs final candidate selection (--score_thresh 0.75)) — https://github.com/zmahnoor14/MAW
- **MAW-R** (R segment performing MS2 preprocessing and spectral/compound database dereplication that feeds candidate ranking) — https://github.com/zmahnoor14/MAW

## Examples

```
docker run -it zmahnoor/maw-py:1.0.7 python Workflow_Python_Script_all_MetFrag.py --score_thresh 0.75
```

## Evaluation signals

- file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one final candidate per feature
- Recomputed coverage percentages match Table 4: MAW structural 100%, chemical class 97.3%, formula 100%; Spectra 75.7%; SIRIUS 70.2%; MetFrag CLI 2.5.0 with PubChem 32.4% (12/37 features)
- All retained MetFrag candidates pass the 0.75 score threshold, and MAW-defined metrics each reach at least 0.5 in addition to high cosine similarity
- Target cyclic bis-bibenzyls rank within the top 25 candidates at expected ranks: Perrottetin E (10), Marchantin H (10), Marchantin M (2), Marchantin D (2), Marchantin F (8)
- Structural matching between spectral and in silico candidates is supported by Tanimoto similarity scores rather than score-free rank ordering

## Limitations

- Larger databases such as PubChem can lead to a larger list of candidates with the actual structure being at a lower rank (e.g. top 100th candidate)
- The MAW-defined 0.5 threshold reduces false positives but can miss candidates with low cosine similarity scores as false negatives
- Public spectral databases are not completely standardized and can misannotate (e.g. betaine annotated as reduced-state trimethylglycine)
- SIRIUS CLI provides no deterministic probability score, so a fixed ranking threshold cannot be applied uniformly; lipid class annotation via SIRIUS can also produce false positives
- Only SIRIUS was integrated for compound databases in the first MAW version; MetFrag integration came later

## Evidence

- [discussion] MAW-defined metrics threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [methods] MetFrag score threshold parameter: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [abstract] Ranking improvement claim: "MAW can improve candidate ranking by integrating spectral databases with annotation tools like SIRIUS"
- [readme] Final candidate selection tooling: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)"
- [discussion] Structural matching scoring scheme: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [discussion] Large-database ranking caveat: "Larger databases such as PubChem can lead to a larger list of candidates with the actual structure being at a lower rank"
