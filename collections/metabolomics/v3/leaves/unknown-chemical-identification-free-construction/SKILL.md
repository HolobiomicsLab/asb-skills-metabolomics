---
name: unknown-chemical-identification-free-construction
description: Use when you have a real/observed mzML file (e.g. output of a prior Top-N
  run) and want ViMMS 2.0 chemical input derived from empirically detected peaks rather
  than synthetically sampled compounds from HMDB.
when_to_use_negative:
- You need fully specified chemicals with known identities/formulae for simulation
  — use ChemicalMixtureCreator with HMDB sampling (hmdb_compounds.p) instead
- Your input is not mzML scan data (e.g. already a pickled chemical list, MGF spectra,
  or an MZmine aligned feature table like multibeers_2_aligned.csv)
- You want a fast, low-memory simulation and cannot afford the high object counts
  produced by low min_intensity thresholds
tools:
- name: ViMMS 2.0
  role: Provides ChemicalMixtureFromMZML, the ROI/FeatureExtraction machinery, and
    save_obj for pickling the UnknownChemical list
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language used to drive the extraction, sampling and pickling workflow (Python
    3.0 or higher)
- name: numpy
  role: Common dependency of ViMMS used for numeric handling of ROI/peak data
- name: pandas
  role: Common dependency of ViMMS used alongside numpy/scipy in the framework
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:b78e0605f83a3af0792af2077a2b61852799dbd102816e84897984e67c8fc11b
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Unknown chemical identification-free construction

## Summary

Builds a chemical dataset for ViMMS 2.0 simulation by extracting ROIs from an existing mzML file (e.g. Beer_multibeers_1_T10_POS.mzML) via ChemicalMixtureFromMZML and converting each picked ROI into an UnknownChemical object, avoiding synthetic sampling from databases like HMDB.

## When to use

Use when you have a real/observed mzML file (e.g. output of a prior Top-N run) and want ViMMS 2.0 chemical input derived from empirically detected peaks rather than synthetically sampled compounds from HMDB.

## When NOT to use

- You need fully specified chemicals with known identities/formulae for simulation — use ChemicalMixtureCreator with HMDB sampling (hmdb_compounds.p) instead
- Your input is not mzML scan data (e.g. already a pickled chemical list, MGF spectra, or an MZmine aligned feature table like multibeers_2_aligned.csv)
- You want a fast, low-memory simulation and cannot afford the high object counts produced by low min_intensity thresholds

## Inputs

- One or more mzML files of previously acquired LC-MS/MS data (e.g. Beer_multibeers_1_T10_POS.mzML)
- ROI extraction parameters (min_intensity threshold, ms_levels=2, n_chemicals=None)

## Outputs

- A list of UnknownChemical objects per input mzML file, with m/z, RT and chromatogram attributes populated from picked ROIs
- A pickled chemical list consumable by ViMMS 2.0 controllers and the Environment

## How to apply

Load the source mzML (e.g. Beer_multibeers_1_T10_POS.mzML from the upstream Top-N acquisition) and instantiate ChemicalMixtureFromMZML, which extracts ROIs from the file's scans using the ROI/FeatureExtraction machinery. Each picked ROI is converted into an UnknownChemical — a putative chemical with unidentified properties carrying m/z, RT and chromatogram attributes populated from the ROI. Call sample() with ms_levels=2 and n_chemicals=None to retain all extracted ROIs rather than subsampling. Pickle the resulting list (e.g. with save_obj) so downstream controllers and the Environment can consume it as an alternative chemical source. Choose the ROI intensity threshold deliberately: the article notes min_intensity=1.75E5 is set very high for fast runs, 1E4 is more accurate but slower, and 0 keeps all ROIs including noise.

## Related tools

- **ViMMS 2.0** (Provides ChemicalMixtureFromMZML, the ROI/FeatureExtraction machinery, and save_obj for pickling the UnknownChemical list) — https://github.com/glasgowcompbio/vimms
- **Python** (Language used to drive the extraction, sampling and pickling workflow (Python 3.0 or higher))
- **numpy** (Common dependency of ViMMS used for numeric handling of ROI/peak data)
- **pandas** (Common dependency of ViMMS used alongside numpy/scipy in the framework)

## Examples

```
from vimms.Chemicals import ChemicalMixtureFromMZML; cm = ChemicalMixtureFromMZML(mzml_file); chemicals = cm.sample(ms_levels=2, n_chemicals=None); save_obj(chemicals, 'chemicals_from_mzml.p')
```

## Evaluation signals

- The output pickle loads successfully (e.g. via load_obj) and is a list of UnknownChemical objects
- Each UnknownChemical has m/z, RT and chromatogram attributes populated from an extracted ROI (non-None, within the mzML's scan range)
- With n_chemicals=None and ms_levels=2, the list length equals the number of picked ROIs in the source mzML at the chosen min_intensity threshold
- The chemical list can be passed to a ViMMS controller running inside an Environment without type/attribute errors

## Limitations

- All chemicals are UnknownChemical objects: peaks are treated as putative chemicals with unidentified properties, so no known identities, formulae or fragmentation trees are available
- min_intensity threshold trades fidelity for speed: 1.75E5 was set very high to complete quickly, 1E4 is more accurate but 'much slower at higher memory usage', and 0 includes noise ROIs
- The paper itself presents no validation of ViMMS 2.0; utility validation is deferred to Davies et al. (2021)

## Evidence

- [abstract] chemicals from mzML files: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"
- [results] Beer mzML input file: "mzml_file = os.path.join(base_dir, 'Beer_multibeers_1_T10_POS.mzML')"
- [results] ROI threshold trade-off: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower"
- [readme] replaying existing data: [evidence span withheld — anchor status: external_doc]
