---
name: ms1-ms2-feature-result-merging
description: Use when you have SIRIUS (.json) outputs and spectral dereplication results
  (spectral_results.csv) for .mzML-derived features and need to combine them with
  the MS1DATA.csv feature table to obtain ranked final candidates per feature.
when_to_use_negative:
- You only have raw .mzML files with no upstream MAW-R spectral dereplication or SIRIUS
  outputs yet — run the R and SIRIUS segments first.
- You need to annotate features absent from public spectral databases with large compound
  databases (e.g. PubChem), where the true structure may be pushed to low ranks (e.g.
  top 100th candidate) rather than relying on merged top-candidate ranking.
- You need MetFrag-based compound annotation — the first MAW version integrates only
  SIRIUS (CSI:FingerID) for compound databases, so this SIRIUS-specific merging script
  is not applicable.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAW-Py
  role: Python segment script (Workflow_Python_Script_all_SIRIUS.py) that post-processes
    and merges spectral database, SIRIUS, and MS1 results into final per-feature candidate
    rankings using RDKit and PubChemPy
  repo: https://github.com/zmahnoor14/MAW
- name: RDKit
  role: Cheminformatics tool used in the final candidate selection for Tanimoto structural
    similarity scoring during merging/prioritization
  repo: https://www.rdkit.org/
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) in which the merging script is executed
  repo: https://github.com/zmahnoor14/MAW
- name: SIRIUS
  role: Upstream compound database annotation tool whose CSI:FingerID scores and .json
    outputs feed the merge
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: Spectra
  role: R package whose cosine similarity metrics (GNPS-altered for GNPS, unaltered
    for HMDB/MassBank) produce the spectral match scores merged into the candidate
    table
  repo: https://rformassspectrometry.github.io/Spectra/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:ef6e91bf1b0802a306762a93289d95805b16038f0fbf626386e03c2dee08db0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - RDKit
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  - maw-sirius5-old
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ms1-ms2-feature-result-merging

## Summary

Post-processing step of the MAW metabolome annotation workflow that merges MS2 spectral/compound dereplication candidate lists with MS1 feature tables into a single per-feature candidate ranking (file_id_mergedResults-with-one-Candidates.csv), executed in the maw-py Docker container.

## When to use

Use when you have SIRIUS (.json) outputs and spectral dereplication results (spectral_results.csv) for .mzML-derived features and need to combine them with the MS1DATA.csv feature table to obtain ranked final candidates per feature.

## When NOT to use

- You only have raw .mzML files with no upstream MAW-R spectral dereplication or SIRIUS outputs yet — run the R and SIRIUS segments first.
- You need to annotate features absent from public spectral databases with large compound databases (e.g. PubChem), where the true structure may be pushed to low ranks (e.g. top 100th candidate) rather than relying on merged top-candidate ranking.
- You need MetFrag-based compound annotation — the first MAW version integrates only SIRIUS (CSI:FingerID) for compound databases, so this SIRIUS-specific merging script is not applicable.

## Inputs

- spectral_results.csv (spectral dereplication output from MAW-R)
- MS1DATA_SiriusP.tsv / MS1DATA.csv (MS1 feature table fed to SIRIUS)
- SIRIUS5 .json output files (run with 'orbitrap' profile and db='ALL')
- GNPS, HMDB, and MassBank dereplication directories (gnps.rda, hmdb.rda, mbankNIST.rda-derived results)
- MS2 .mzML input data files (upstream)

## Outputs

- file_id_mergedResults-with-one-Candidates.csv containing merged candidate names and final MAW ranks per feature
- Per-feature prioritized final candidate list importable to a chemical structure similarity network

## How to apply

After the R segment (MAW-R via the maw-r container) has produced spectral_results.csv and the SIRIUS input table MS1DATA_SiriusP.tsv, and SIRIUS5 has run in the maw-sirius5-old container with the 'orbitrap' profile and db='ALL', run Workflow_Python_Script_all_SIRIUS.py in the maw-py container (zmahnoor/maw-py:1.0.7, --platform=linux/amd64). Pass --msp_file pointing to spectral_results.csv, the GNPS/HMDB/MassBank dereplication directories, --ms1data pointing to MS1DATA.csv, and --db set to the same database used in SIRIUS (e.g. 'ALL'). The script merges candidates, applies candidate selection scoring (cosine similarity from Spectra, SIRIUS/CSI:FingerID scores, and Tanimoto structural similarity via RDKit; MAW-defined metrics should reach at least a 0.5 matching score in addition to a high cosine similarity), and prioritizes one final candidate per feature. Validate by inspecting file_id_mergedResults-with-one-Candidates.csv for per-feature candidate names and MAW ranks.

## Related tools

- **MAW-Py** (Python segment script (Workflow_Python_Script_all_SIRIUS.py) that post-processes and merges spectral database, SIRIUS, and MS1 results into final per-feature candidate rankings using RDKit and PubChemPy) — https://github.com/zmahnoor14/MAW
- **RDKit** (Cheminformatics tool used in the final candidate selection for Tanimoto structural similarity scoring during merging/prioritization) — https://www.rdkit.org/
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) in which the merging script is executed) — https://github.com/zmahnoor14/MAW
- **SIRIUS** (Upstream compound database annotation tool whose CSI:FingerID scores and .json outputs feed the merge) — https://bio.informatik.uni-jena.de/software/sirius/
- **Spectra** (R package whose cosine similarity metrics (GNPS-altered for GNPS, unaltered for HMDB/MassBank) produce the spectral match scores merged into the candidate table) — https://rformassspectrometry.github.io/Spectra/

## Examples

```
python Workflow_Python_Script_all_SIRIUS.py --msp_file spectral_results.csv --ms1data MS1DATA.csv --db ALL
```

## Evaluation signals

- file_id_mergedResults-with-one-Candidates.csv exists and contains candidate names plus MAW ranks for each feature.
- On the nine diatom standards dataset, 6 of 9 standards appear as rank-1 candidates, isovaleryl-l-carnitine is ranked 2, and cysteinolic acid is ranked 3 — reproducing the reported benchmark.
- Betaine is expected to appear only as its reduced form trimethylglycine, reflecting known public spectral database limitations rather than a merge bug.
- MAW-defined metrics in the merged table meet at least a 0.5 matching score in addition to high cosine similarity.
- The merged results are reproducible and traceable across re-runs of the pinned Docker images (maw-r:1.0.8, maw-sirius5-old:1.0.0, maw-py:1.0.7).

## Limitations

- SIRIUS CLI does not provide a deterministic probability score, so candidate ranking thresholds were discarded; thresholds differ from case to case.
- Large databases such as PubChem can produce larger candidate lists with the actual structure at a low rank (e.g. top 100th candidate).
- Public spectral databases are not completely standardized, so features absent from the databases (e.g. betaine) cannot be correctly annotated in the merged results.
- Only SIRIUS (CSI:FingerID) is integrated for compound databases; MetFrag-based merging is not available in the first MAW version.
- The MAW-defined 0.5 threshold can lower false positives but may miss candidates with low cosine similarity as false negatives.

## Evidence

- [abstract] The final candidate selection is performed using the cheminformatics tool RDKit in the Python segment (MAW-Py): "The final candidate selection is performed using the cheminformatics tool RDKit in the Python segment (MAW-Py)"
- [intro] MAW provides top-scoring candidates as a network and prioritizes a final candidate per feature: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
- [discussion] MAW-defined metrics threshold of 0.5 matching score: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [discussion] Large databases can push the true structure to low ranks: "Larger databases such as PubChem can lead to a larger list of candidates with the actual structure being at a lower rank (e.g.: top 100th candidate)"
- [readme] MAW-Py invocation parameters from the README: "Final candidate selection is done in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/)."
