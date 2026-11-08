---
name: chemical-list-serialization-pickle
description: Use when a chemical list extracted from an mzML file (e.g. Beer_multibeers_1_T10_POS.mzML)
  or sampled synthetically must be persisted between ViMMS runs so controllers and
  the Environment can consume it as an alternative chemical source.
when_to_use_negative:
- The chemicals are intended as a shareable, language-agnostic artifact — pickle is
  Python-specific; use an interchange format instead.
- The chemical set is already available as a persisted pickle from a previous run
  — reload it rather than re-extracting ROIs from the mzML.
- The input is not mzML (e.g. aligned MZMine CSVs like multibeers_2_aligned.csv) —
  ChemicalMixtureFromMZML expects mzML scans for ROI extraction.
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: ViMMS 2.0
  role: Provides ChemicalMixtureFromMZML for ROI extraction into UnknownChemical objects
    and save_obj for pickling the resulting chemical list
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Runtime for executing the extraction and pickle serialization workflow
  repo: https://github.com/glasgowcompbio/vimms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:756c22c34bf527683151dd9d3efb2e3973d1375fa8d9e5e06365d72df91a1188
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chemical-list-serialization-pickle

## Summary

Persist the list of UnknownChemical objects produced by ViMMS 2.0's ChemicalMixtureFromMZML ROI extraction (or synthetic sampling) to a pickle file via save_obj, so the chemical list can be reloaded and consumed by downstream controllers and the Environment.

## When to use

Use when a chemical list extracted from an mzML file (e.g. Beer_multibeers_1_T10_POS.mzML) or sampled synthetically must be persisted between ViMMS runs so controllers and the Environment can consume it as an alternative chemical source.

## When NOT to use

- The chemicals are intended as a shareable, language-agnostic artifact — pickle is Python-specific; use an interchange format instead.
- The chemical set is already available as a persisted pickle from a previous run — reload it rather than re-extracting ROIs from the mzML.
- The input is not mzML (e.g. aligned MZMine CSVs like multibeers_2_aligned.csv) — ChemicalMixtureFromMZML expects mzML scans for ROI extraction.

## Inputs

- Source mzML file (e.g. Beer_multibeers_1_T10_POS.mzML from a prior acquisition/simulation)
- ChemicalMixtureFromMZML instance configured with ms_levels=2 and n_chemicals=None to retain all ROIs
- List of UnknownChemical objects returned by sample()

## Outputs

- Pickle file containing the list of UnknownChemical objects (persisted with save_obj)
- Reloadable chemical dataset consumable by ViMMS controllers and the Environment

## How to apply

Instantiate ChemicalMixtureFromMZML on the source mzML file (e.g. the Beer_multibeers_1_T10_POS.mzML output of an upstream Top-N run), which extracts ROIs from the file's scans via the ROI/FeatureExtraction machinery and converts each picked ROI into an UnknownChemical with unidentified properties. Call sample() with ms_levels=2 and n_chemicals=None to keep all extracted ROIs. Then serialize the resulting list with save_obj (vimms.Common) to a pickle file. Downstream, reload the pickle so controllers and the Environment treat these UnknownChemicals as the chemical dataset, as an alternative to chemicals produced via ChemicalMixtureCreator. Note the min_intensity threshold governs how many ROIs become chemicals: 1.75E5 was used in the paper for speed, 1E4 for a more accurate but slower simulation, and 0 includes even noise ROIs.

## Related tools

- **ViMMS 2.0** (Provides ChemicalMixtureFromMZML for ROI extraction into UnknownChemical objects and save_obj for pickling the resulting chemical list) — https://github.com/glasgowcompbio/vimms
- **Python** (Runtime for executing the extraction and pickle serialization workflow) — https://github.com/glasgowcompbio/vimms

## Examples

```
from vimms.ChemicalMixtureFromMZML import ChemicalMixtureFromMZML; from vimms.Common import save_obj; chemicals = ChemicalMixtureFromMZML(mzml_file).sample(ms_levels=2, n_chemicals=None); save_obj(chemicals, 'chemicals.p')
```

## Evaluation signals

- The saved pickle reloads successfully (e.g. via load_obj) without errors.
- Every reloaded entry is an UnknownChemical object.
- Each UnknownChemical has m/z, RT and chromatogram attributes populated from the extracted ROIs.
- With n_chemicals=None, the number of persisted chemicals equals the number of extracted ROIs for the given min_intensity threshold.
- The reloaded list can be passed to a controller run inside an Environment without type or attribute errors.

## Limitations

- Pickle output is Python/ViMMS-specific and not a portable interchange format.
- The number of persisted chemicals depends on the ROI intensity threshold: 1.75E5 (fast, paper default), 1E4 (more accurate but slower, higher memory), or 0 (all ROIs including noise).
- UnknownChemical entries are putative chemicals with unidentified properties, not database-annotated compounds like those from HMDB sampling.

## Evidence

- [methods] ChemicalMixtureFromMZML produces UnknownChemical list persistable via save_obj: [evidence span withheld — anchor status: not_found]
- [abstract] ViMMS generates chemical objects from existing mzML files: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [results] Paper uses Beer_multibeers_1_T10_POS.mzML as the source mzML: "mzml_file = os.path.join(base_dir, 'Beer_multibeers_1_T10_POS.mzML')"
- [results] min_intensity threshold controls ROI count and simulation speed: "Setting the threshold to 0 means using all ROIs as simulation input, including even the noise"
