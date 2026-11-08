---
name: metabolite-candidate-ranking
description: Use when untargeted LC-MS2 .mzML features need prioritized candidate
  metabolites with per-feature ranks — e.g., benchmarking datasets with known standards
  or annotating features from natural-product extract datasets.
when_to_use_negative:
- You need a deterministic probabilistic ranking score for thresholding — SIRIUS CLI
  provides no deterministic probability score, so MAW discards candidate-ranking thresholds
- The target spectra are absent from public spectral databases — the mismatch between
  experimental spectra in spectral databases and structures in compound databases
  leads to few annotated features
- You are searching very large compound databases (e.g., PubChem) where the actual
  structure can be pushed to low ranks (e.g., top 100th candidate), degrading top-k
  ranking
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Spectra
  role: R package performing spectral database dereplication against GNPS, HMDB, and
    MassBank via cosine similarity
  repo: https://rformassspectrometry.github.io/Spectra/
- name: SIRIUS
  role: In silico compound database dereplication (CSI:FingerID) with db 'ALL' and
    'orbitrap' profile inside the maw-sirius5-old container
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: MetFrag
  role: Alternative in silico compound annotation against COCONUT; results filtered
    at a score threshold of 0.75
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: RDKit
  role: Final candidate selection in MAW-Py, including Tanimoto similarity for structural
    matching
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python segment that post-processes and merges candidates from spectral_results.csv,
    dereplication directories, and MS1DATA.csv into the mergedResults table
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-R
  role: R segment that preprocesses .mzML input and performs spectral database dereplication
    at 15 ppm to generate the SIRIUS input table
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:72261c4c6efb3ffae9c654d845a136e8685b9f150c44bf5bbd69d32418d5e614
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
  - maw-sirius5-old
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 24
---

# metabolite-candidate-ranking

## Summary

Ranks putative metabolite candidates per MS2 feature by merging spectral database dereplication (Spectra against GNPS/HMDB/MassBank) with in silico annotation (SIRIUS or MetFrag) and RDKit-based final candidate selection, as implemented in the MAW workflow.

## When to use

Use when untargeted LC-MS2 .mzML features need prioritized candidate metabolites with per-feature ranks — e.g., benchmarking datasets with known standards or annotating features from natural-product extract datasets.

## When NOT to use

- You need a deterministic probabilistic ranking score for thresholding — SIRIUS CLI provides no deterministic probability score, so MAW discards candidate-ranking thresholds
- The target spectra are absent from public spectral databases — the mismatch between experimental spectra in spectral databases and structures in compound databases leads to few annotated features
- You are searching very large compound databases (e.g., PubChem) where the actual structure can be pushed to low ranks (e.g., top 100th candidate), degrading top-k ranking

## Inputs

- MS2 .mzML data files from untargeted LC-MS/MS
- Spectral database objects gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)
- COCONUT CSV compound database for MetFrag runs (Zenodo record 7704937)
- spectral_results.csv from MAW-R spectral dereplication
- MS1DATA.csv / MS1DATA_SiriusP.tsv feature tables

## Outputs

- spectral_results.csv (spectral dereplication per feature)
- MS1DATA_SiriusP.tsv (SIRIUS input) and SIRIUS .json outputs
- file_id_mergedResults-with-one-Candidates.csv with final candidate names, SMILES, and MAW ranks per feature
- chemical class annotations per candidate (e.g., CANOPUS/Flavonoids class with MSI levels)

## How to apply

Run MAW-R on the .mzML files with the gnps.rda, hmdb.rda, and mbankNIST.rda spectral database objects at 15 ppm precursor-mass tolerance (collision energy enabled) to produce spectral_results.csv and the MS1DATA_SiriusP.tsv SIRIUS input table. Run SIRIUS5 with the 'orbitrap' profile and db set to 'ALL' (or, for the MetFrag variant, annotate against COCONUT and filter with a MetFrag score threshold of 0.75). In the MAW-Py container, run Workflow_Python_Script_all_SIRIUS.py (or _all_MetFrag.py) with --msp_file spectral_results.csv and --ms1data MS1DATA.csv so RDKit merges and prioritizes one final candidate per feature using cosine similarity, the MAW-defined metrics (each ≥ 0.5 plus high cosine similarity), and the Tanimoto similarity score for structural matching. Inspect file_id_mergedResults-with-one-Candidates.csv for final candidate names and MAW ranks per feature.

## Related tools

- **Spectra** (R package performing spectral database dereplication against GNPS, HMDB, and MassBank via cosine similarity) — https://rformassspectrometry.github.io/Spectra/
- **SIRIUS** (In silico compound database dereplication (CSI:FingerID) with db 'ALL' and 'orbitrap' profile inside the maw-sirius5-old container) — https://bio.informatik.uni-jena.de/software/sirius/
- **MetFrag** (Alternative in silico compound annotation against COCONUT; results filtered at a score threshold of 0.75) — https://github.com/ipb-halle/MetFragRelaunched
- **RDKit** (Final candidate selection in MAW-Py, including Tanimoto similarity for structural matching) — https://www.rdkit.org/
- **MAW-Py** (Python segment that post-processes and merges candidates from spectral_results.csv, dereplication directories, and MS1DATA.csv into the mergedResults table) — https://github.com/zmahnoor14/MAW
- **MAW-R** (R segment that preprocesses .mzML input and performs spectral database dereplication at 15 ppm to generate the SIRIUS input table) — https://github.com/zmahnoor14/MAW

## Examples

```
docker run -it --platform=linux/amd64 -v $(pwd):/opt/workdir/data zmahnoor/maw-py:1.0.7 python Workflow_Python_Script_all_SIRIUS.py --msp_file spectral_results.csv --ms1data MS1DATA.csv --db ALL
```

## Evaluation signals

- On the nine diatom standards, 6 of 9 standards are rank-1 candidates, isovaleryl-l-carnitine is ranked 2, and cysteinolic acid is ranked 3 in the merged results
- file_id_mergedResults-with-one-Candidates.csv exists and contains exactly one prioritized final candidate per feature
- All MAW-defined metrics for a candidate have at least a 0.5 matching score in addition to a high cosine similarity score
- For bryophyte feature M287R318, the record reports Nicotiflorin with formula C27H30O15, Flavonoids class, and spectral matches to all three databases (MSI-level 2)
- MetFrag-mode runs apply the --score_thresh 0.75 filter before RDKit-based final selection

## Limitations

- SIRIUS CLI does not provide a deterministic probability score, so candidate-ranking thresholds were discarded as case-dependent
- Public spectral databases are not completely standardized — e.g., betaine was only identified as its reduced form, trimethylglycine
- The MAW-defined 0.5 threshold reduces false positives but can miss candidates with low cosine similarity scores as false negatives
- Larger databases such as PubChem can lead to longer candidate lists with the actual structure at a lower rank
- Only SIRIUS (CSI:FingerID) was integrated for compound database annotation in the first MAW version

## Evidence

- [abstract] MAW can improve candidate ranking by integrating spectral databases with annotation tools like SIRIUS: "MAW can improve candidate ranking by integrating spectral databases with annotation tools like SIRIUS"
- [methods] '--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)': "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [readme] It performs spectral database dereplication using R Package [Spectra](https://rformassspectrometry.github.io/Spectra/) and compound database dereplication using [SIRIUS]: "It performs spectral database dereplication using R Package [Spectra](https://rformassspectrometry.github.io/Spectra/)"
- [discussion] SIRIUS CLI doesn't provide a deterministic probability score, as a result, providing a threshold would differ from case to case, and hence the idea was discarded: "SIRIUS CLI doesn't provide a deterministic probability score, as a result, providing a threshold would differ from case to case, and hence the idea was discarded"
