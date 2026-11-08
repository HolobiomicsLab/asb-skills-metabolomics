---
name: insilico-msms-annotation-sirius
description: Use when MS2 .mzML features lack spectral-library matches and candidate
  metabolite structures must be ranked via SIRIUS in silico fragmentation with the
  'all' (orbitrap) database inside the MAW containers.
when_to_use_negative:
- The feature already has a confident spectral database dereplication hit — MAW's
  spectral matching (e.g. cosine similarity against gnps.rda/hmdb.rda/mbankNIST.rda)
  may suffice, and SIRIUS 'bio'/'ALL' database rankings can place true structures
  lower
- 'Annotating lipid classes: SIRIUS lipid class annotation can produce false positives
  due to co-eluting lipid isomers'
- Targets are macrocyclic natural products (e.g. Marchantin) whose complex fragmentation
  makes in silico annotation unreliable — suspect lists are recommended instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Compound database dereplication / in silico annotation via CSI:FingerID, run
    with 'orbitrap' profile and db 'ALL' in the maw-sirius5-old container
  repo: https://github.com/zmahnoor14/MAW
- name: maw-sirius5-old
  role: Docker image (zmahnoor/maw-sirius5-old:1.0.0, linux/amd64) that executes Run_Sirius.r
    on the MAW-R-generated input table
- name: MAW-Py
  role: Python segment (Workflow_Python_Script_all_SIRIUS.py) merging SIRIUS outputs
    with spectral dereplication and performing final candidate selection via RDKit
  repo: https://github.com/zmahnoor14/MAW
- name: Spectra
  role: R package performing spectral database dereplication (cosine similarity) upstream
    of SIRIUS annotation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:c8d9b9124c28c6ded908c1406061feea3d86383c0d370e4f7a4125756a016ff4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# insilico-msms-annotation-sirius

## Summary

In silico MS/MS candidate annotation using SIRIUS (CSI:FingerID) within the MAW workflow, as demonstrated by benchmarking SIRIUS 'all' database rankings against nine known diatom standard metabolites.

## When to use

Use when MS2 .mzML features lack spectral-library matches and candidate metabolite structures must be ranked via SIRIUS in silico fragmentation with the 'all' (orbitrap) database inside the MAW containers.

## When NOT to use

- The feature already has a confident spectral database dereplication hit — MAW's spectral matching (e.g. cosine similarity against gnps.rda/hmdb.rda/mbankNIST.rda) may suffice, and SIRIUS 'bio'/'ALL' database rankings can place true structures lower
- Annotating lipid classes: SIRIUS lipid class annotation can produce false positives due to co-eluting lipid isomers
- Targets are macrocyclic natural products (e.g. Marchantin) whose complex fragmentation makes in silico annotation unreliable — suspect lists are recommended instead

## Inputs

- MS/MS peaktable in .mzML format (e.g. diatom standards dataset)
- SIRIUS input table MS1DATA_SiriusP.tsv with .ms files under insilico/SIRIUS/no_isotope (produced by MAW-R)
- Spectral database objects gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)
- Spectral dereplication results spectral_results.csv

## Outputs

- SIRIUS5 per-feature annotation .json files
- Merged candidate table file_id_mergedResults-with-one-Candidates.csv with candidate names and MAW ranks per feature
- Chemical structure assignments per feature importable to a chemical structure similarity network

## How to apply

Run the MAW-R container first to produce spectral dereplication results (spectral_results.csv) and the SIRIUS input table MS1DATA_SiriusP.tsv with .ms files under insilico/SIRIUS/no_isotope, using a 15 ppm precursor tolerance against the gnps.rda, hmdb.rda and mbankNIST.rda databases from Zenodo record 7519270. Then, in the maw-sirius5-old container, authenticate with 'sirius login' and run Run_Sirius.r on the MS1DATA_SiriusP.tsv file with the 'orbitrap' instrument profile and db set to 'ALL' so SIRIUS5 writes per-feature .json outputs. Finally, post-process in the maw-py container with Workflow_Python_Script_all_SIRIUS.py, merging SIRIUS candidates with spectral dereplication results; note that the SIRIUS CLI provides no deterministic probability score, so candidate-ranking thresholds are intentionally not applied at this stage. Validate the ranking behaviour against known standards (e.g. 6 of 9 diatom standards rank 1; isovaleryl-l-carnitine rank 2; cysteinolic acid rank 3), and remember that spectral database metrics (Spectra cosine similarity plus MAW-defined metrics with a 0.5 threshold) and RDKit Tanimoto structural scoring in MAW-Py supply the final candidate prioritization that SIRIUS alone does not.

## Related tools

- **SIRIUS** (Compound database dereplication / in silico annotation via CSI:FingerID, run with 'orbitrap' profile and db 'ALL' in the maw-sirius5-old container) — https://github.com/zmahnoor14/MAW
- **maw-sirius5-old** (Docker image (zmahnoor/maw-sirius5-old:1.0.0, linux/amd64) that executes Run_Sirius.r on the MAW-R-generated input table)
- **MAW-Py** (Python segment (Workflow_Python_Script_all_SIRIUS.py) merging SIRIUS outputs with spectral dereplication and performing final candidate selection via RDKit) — https://github.com/zmahnoor14/MAW
- **Spectra** (R package performing spectral database dereplication (cosine similarity) upstream of SIRIUS annotation)

## Examples

```
docker run --platform=linux/amd64 zmahnoor/maw-sirius5-old:1.0.0 sirius login && Rscript Run_Sirius.r /opt/workdir/data/your_file_name/insilico/MS1DATA_SiriusP.tsv orbitrap ALL
```

## Evaluation signals

- Merged results table file_id_mergedResults-with-one-Candidates.csv reproduces benchmark behaviour: 6 of 9 diatom standards are rank-1 candidates, isovaleryl-l-carnitine ranked 2, cysteinolic acid ranked 3
- Betaine identified as its reduced form trimethylglycine (8 of 9 standards correctly identified at reduced-form level), matching reported database-standardization limits
- SIRIUS .json outputs are produced for every feature row in MS1DATA_SiriusP.tsv run with db 'ALL'
- MAW-defined metrics (cosine similarity plus companion scores) each reach at least 0.5 matching score in the merged candidate list

## Limitations

- SIRIUS CLI provides no deterministic probability score, so candidate ranking thresholds would differ case to case and were discarded
- Large databases (db 'ALL'; e.g. PubChem-scale) can push the true structure to a low rank such as top 100th candidate
- Using SIRIUS 'bio' database ranked actual bryophyte compounds lower than COCONUT
- SIRIUS lipid class annotation can yield false positives due to co-eluting lipid isomers
- Only SIRIUS (CSI:FingerID) is integrated for compound databases in MAW v1; MetFrag integration is pending

## Evidence

- [methods] SIRIUS run profile and database settings: [evidence span withheld — anchor status: not_found]
- [discussion] CLI threshold limitation: "SIRIUS CLI doesn't provide a deterministic probability score, as a result, providing a threshold would differ from case to case, and hence the idea was discarded"
- [discussion] Large-database ranking caveat: "Larger databases such as PubChem can lead to a larger list of candidates with the actual structure being at a lower rank (e.g.: top 100th candidate)"
- [results] Benchmark validation behaviour: [evidence span withheld — anchor status: not_found]
- [readme] README on SIRIUS role: "It performs spectral database dereplication using R Package [Spectra](https://rformassspectrometry.github.io/Spectra/) and compound database dereplication using [SIRIUS]"
