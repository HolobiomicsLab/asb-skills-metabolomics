---
name: cheminformatics-candidate-ranking
description: Use when an untargeted MS2 feature (mzML) has a shortlist of putative
  candidates from spectral/compound database dereplication and one final prioritized
  structure per feature is needed.
when_to_use_negative:
- The input is already a ranked, validated feature annotation table — re-ranking adds
  no value.
- The target compounds are macrocyclic or highly complex structures (e.g. Marchantin)
  where complex fragmentation patterns make in silico candidate ranking unreliable
  — suspect lists are recommended instead.
- Only formula or class-level annotation is achievable (e.g. SIRIUS lipid class annotations
  prone to false positives) — structural candidate ranking is not appropriate.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3336
tools:
- name: RDKit
  role: Performs the final candidate selection via structural matching (Tanimoto similarity)
    in the Python segment
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python workflow segment that integrates RDKit-based ranking and candidate
    prioritization per feature
  repo: https://github.com/zmahnoor14/MAW
- name: SIRIUS
  role: Provides in silico fragmentation candidate scores (CSI:FingerID) used together
    with Spectra scoring for ranking
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: Spectra
  role: Supplies the cosine similarity metric for spectral database matching that
    feeds candidate ranking
  repo: https://rformassspectrometry.github.io/Spectra/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:0520d72701a9b645389b7f88b2ac26ccb2e7ac8261a46c7a28044d80f0107aa6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-Py
  - RDKit
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cheminformatics-candidate-ranking

## Summary

Rank and prioritize a final chemical structure candidate per metabolomics feature by combining spectral similarity (cosine from Spectra), in silico fragmentation scores (SIRIUS/CSI:FingerID), and structural Tanimoto matching with RDKit in the MAW-Py segment of the workflow.

## When to use

Use when an untargeted MS2 feature (mzML) has a shortlist of putative candidates from spectral/compound database dereplication and one final prioritized structure per feature is needed.

## When NOT to use

- The input is already a ranked, validated feature annotation table — re-ranking adds no value.
- The target compounds are macrocyclic or highly complex structures (e.g. Marchantin) where complex fragmentation patterns make in silico candidate ranking unreliable — suspect lists are recommended instead.
- Only formula or class-level annotation is achievable (e.g. SIRIUS lipid class annotations prone to false positives) — structural candidate ranking is not appropriate.

## Inputs

- Per-feature candidate lists from spectral database dereplication (MAW-R outputs: msp file, gnps_dir, hmdb_dir, mbank_dir results)
- SIRIUS output (MS1DATA_SiriusP.tsv results from maw-sirius5-old container)
- Compound/spectral databases (gnps.rda, hmdb.rda, mbankNIST.rda from Zenodo record 7519270; COCONUT from Zenodo record 7704937)

## Outputs

- One prioritized final chemical structure candidate per feature
- Ranked candidate list per feature with cosine, SIRIUS/CSI:FingerID, and Tanimoto scores
- Chemical structure assignments importable to a chemical structure similarity network

## How to apply

After MAW-R produces per-feature candidate lists (via Spectra cosine matching against gnps.rda, hmdb.rda, and mbankNIST.rda and SIRIUS runs on MS1DATA_SiriusP.tsv with e.g. ppm_max=5, ppm_max_ms2=15, candidates=30, db=coconut), post-process them in the maw-py container with Workflow_Python_Script_all_SIRIUS.py. Combine the scoring schemes from Spectra and SIRIUS (CSI:FingerID) with the Tanimoto similarity score for structural matching computed by RDKit; MAW-defined metrics should have at least a 0.5 matching score in addition to a high cosine similarity. Apply a score threshold for annotation results (default --score_thresh 0.75 for MetFrag-type results) and prioritize one final candidate per feature. Choose a focused compound database (e.g. COCONUT for natural products) since larger databases such as PubChem can push the actual structure to a low rank (e.g. top 100th candidate). Each feature is then assigned a chemical structure that can be imported into a chemical structure similarity network.

## Related tools

- **RDKit** (Performs the final candidate selection via structural matching (Tanimoto similarity) in the Python segment) — https://www.rdkit.org/
- **MAW-Py** (Python workflow segment that integrates RDKit-based ranking and candidate prioritization per feature) — https://github.com/zmahnoor14/MAW
- **SIRIUS** (Provides in silico fragmentation candidate scores (CSI:FingerID) used together with Spectra scoring for ranking) — https://bio.informatik.uni-jena.de/software/sirius/
- **Spectra** (Supplies the cosine similarity metric for spectral database matching that feeds candidate ranking) — https://rformassspectrometry.github.io/Spectra/

## Examples

```
python3.10 Workflow_Python_Script_all_SIRIUS.py --msp_file maw-r/out.msp --gnps_dir gnps/ --hmdb_dir hmdb/ --mbank_dir mbankNIST/ --ms1data MS1DATA_SiriusP.tsv --db coconut
```

## Evaluation signals

- Each retained candidate satisfies MAW-defined metrics of at least a 0.5 matching score plus a high cosine similarity score.
- MetFrag-type annotation results fall below the --score_thresh threshold (default 0.75) filter.
- Every processed feature ends up with exactly one prioritized final candidate, or is explicitly reported as unannotated (formula-only/class-only).
- Tally of structurally annotated features is reproducible — e.g. 881/933 bryophyte (MTBLS709) features annotated with chemical structures when rerunning the workflow.
- Ranking with a focused database (COCONUT) places the actual compound higher than with broader databases (e.g. SIRIUS 'bio') — compare top ranks across database choices.

## Limitations

- Cosine similarity fails under differing fragmentation patterns or peak shifts; MAW concludes spectral matching needs better metrics.
- The 0.5 threshold reduces false positives but can miss candidates with low cosine similarity scores as false negatives.
- Public spectral databases are not standardized, leading to misannotation (e.g. betaine annotated as reduced-state trimethylglycine); a mismatch between experimental spectra and compound-database structures limits annotations.
- Larger databases such as PubChem can lead to a larger candidate list with the actual structure at a low rank (e.g. top 100th candidate).
- SIRIUS CLI provides no deterministic probability score, so candidate ranking thresholds were discarded for its results.

## Evidence

- [abstract] The final candidate selection is performed using the cheminformatics tool RDKit: "The final candidate selection is performed using the cheminformatics tool RDKit"
- [discussion] Tanimoto scoring for structural matching: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [discussion] MAW-defined metric threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [methods] score_thresh parameter: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] Database choice affects ranking: "Using databases specifically focusing on natural products like COCONUT and GNPS can increase the probability of finding the compound of interest within the top-ranked candidates"
- [readme] README: final selection in Python with RDKit and PubChemPy: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)"
