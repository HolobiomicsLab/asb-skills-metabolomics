---
name: cheminformatics-candidate-structure-selection
description: Use when a candidate list per LC-MS/MS feature exists from spectral and
  compound database dereplication and one structure per feature must be prioritized
  via structural scoring (e.g. Tanimoto similarity on SMILES) in Python.
when_to_use_negative:
- Input is raw MS2 .mzML data that has not yet gone through MAW-R spectral database
  dereplication — run the R segment first to produce spectral_results.csv.
- Only a single annotation tool's output is available with no merged spectral/compound
  candidates — RDKit-based selection in MAW-Py requires the combined candidate lists.
- The goal is spectral-library-only matching against standardized entries — MAW's
  selection thresholds can discard true candidates with low cosine similarity (false
  negatives).
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Performs final candidate selection and structural (Tanimoto) scoring on candidate
    SMILES in the Python segment
  repo: https://www.rdkit.org/
- name: MAW-Py
  role: Python workflow segment (maw-py docker image) that merges candidate lists
    and runs RDKit selection with --score_thresh default 0.75
  repo: https://github.com/zmahnoor14/MAW
- name: MetFrag
  role: Supplies compound-database candidate lists (e.g. against COCONUT) whose scores
    are thresholded and merged before RDKit selection
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: SIRIUS
  role: Alternative compound-database annotation tool whose CSI:FingerID scores feed
    candidate ranking alongside Tanimoto similarity
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: PubChemPy
  role: Used in MAW-Py alongside RDKit during final candidate selection
  repo: https://pubchempy.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:09e103acdaa32c574ed4e2537c74f983d40c9fd5882b2ac3c264d6423c0ab61e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAW-Py
  - MetFrag
  - PubChemPy
  - RDKit
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cheminformatics-candidate-structure-selection

## Summary

Final candidate selection step of the MAW workflow: merge spectral-database dereplication results (GNPS, HMDB, MassBank) with compound-database candidates (SIRIUS or MetFrag), then use RDKit-based cheminformatics scoring in MAW-Py to prioritize one final candidate per MS2 feature.

## When to use

Use when a candidate list per LC-MS/MS feature exists from spectral and compound database dereplication and one structure per feature must be prioritized via structural scoring (e.g. Tanimoto similarity on SMILES) in Python.

## When NOT to use

- Input is raw MS2 .mzML data that has not yet gone through MAW-R spectral database dereplication — run the R segment first to produce spectral_results.csv.
- Only a single annotation tool's output is available with no merged spectral/compound candidates — RDKit-based selection in MAW-Py requires the combined candidate lists.
- The goal is spectral-library-only matching against standardized entries — MAW's selection thresholds can discard true candidates with low cosine similarity (false negatives).

## Inputs

- spectral_results.csv from MAW-R spectral dereplication
- GNPS, HMDB, and MassBank spectral-match result directories
- MetFrag candidate results (against COCONUT CSV from Zenodo record 7704937)
- MS1DATA.csv feature table with precursor m/z and retention times
- candidate SMILES strings for structural scoring

## Outputs

- mergedResults CSV with one prioritized candidate per MS2 feature
- per-feature precursor m/z, RT, molecular formula, chemical class, candidate SMILES, and MSI confidence levels
- chemical structures per feature importable into a chemical structure similarity network

## How to apply

Run MAW-Py (docker image zmahnoor/maw-py:1.0.7) with Workflow_Python_Script_all_MetFrag.py, passing paths to spectral_results.csv, the GNPS/HMDB/MassBank result directories, and MS1DATA.csv, and set the MetFrag score threshold (--score_thresh, default 0.75) to pre-filter candidates. MAW-Py merges the MetFrag/spectral candidates with the MS1 feature table and performs final candidate selection with RDKit, using the Spectra and SIRIUS (CSI:FingerID) scoring schemes plus the Tanimoto similarity score for structural matching between candidate SMILES. Beyond the per-tool scores, MAW-defined metrics (matching score, cosine similarity) should each reach at least 0.5 in addition to a high cosine similarity to retain a candidate. The output is a mergedResults CSV containing one prioritized candidate per feature (e.g. feature file_1M287R318ID319, precursor m/z 287.065, RT 318.265 s resolved to Nicotiflorin, formula C27H30O15, class Flavonoids), which also feeds chemical structure similarity networks and MSI confidence-level reporting.

## Related tools

- **RDKit** (Performs final candidate selection and structural (Tanimoto) scoring on candidate SMILES in the Python segment) — https://www.rdkit.org/
- **MAW-Py** (Python workflow segment (maw-py docker image) that merges candidate lists and runs RDKit selection with --score_thresh default 0.75) — https://github.com/zmahnoor14/MAW
- **MetFrag** (Supplies compound-database candidate lists (e.g. against COCONUT) whose scores are thresholded and merged before RDKit selection) — https://github.com/ipb-halle/MetFragRelaunched
- **SIRIUS** (Alternative compound-database annotation tool whose CSI:FingerID scores feed candidate ranking alongside Tanimoto similarity) — https://bio.informatik.uni-jena.de/software/sirius/
- **PubChemPy** (Used in MAW-Py alongside RDKit during final candidate selection) — https://pubchempy.readthedocs.io/en/latest/

## Examples

```
docker run --platform=linux/amd64 zmahnoor/maw-py:1.0.7 python Workflow_Python_Script_all_MetFrag.py --spectral_results spectral_results.csv --gnps_dir GNPS --hmdb_dir HMDB --mbank_dir MassBank --ms1 MS1DATA.csv --score_thresh 0.75
```

## Evaluation signals

- mergedResults CSV exists and contains a prioritized candidate row for each input MS1 feature (e.g. file_1M287R318ID319 present).
- Retained candidates meet MetFrag score threshold 0.75 and MAW-defined metrics of at least 0.5 matching score with high cosine similarity.
- For the bryophyte validation feature M287R318, the selected candidate is Nicotiflorin with formula C27H30O15, class Flavonoids, and spectral support from GNPS, HMDB, and MassBank (MSI-level 2).
- Each selected candidate carries a valid RDKit-parseable SMILES, enabling import into a chemical structure similarity network.
- MSI confidence levels are consistently assigned per annotation source: precursor/formula at MSI-level 4–5, chemical class at MSI-level 3, spectral-database matches at MSI-level 2.

## Limitations

- MAW-defined 0.5 threshold may miss candidates with low cosine similarity as false negatives.
- Only SIRIUS (CSI:FingerID) was integrated for compound database annotation in the first MAW version; MetFrag integration for compound databases was not yet complete.
- SIRIUS CLI does not provide a deterministic probability score, so candidate ranking thresholds for it were discarded.
- Large databases such as PubChem can push the actual structure to low ranks (e.g. top 100th candidate).

## Evidence

- [abstract] Final candidate selection via RDKit in MAW-Py: "The final candidate selection is performed using the cheminformatics tool RDKit"
- [methods] MetFrag score threshold default: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] Tanimoto scoring in candidate selection: "We used the scoring schemes provided by Spectra and SIRIUS (CSI:FingerID), and the Tanimoto similarity score for the structural matching"
- [discussion] MAW-defined retention thresholds: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [intro] Candidate selection purpose: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
