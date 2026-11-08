---
name: chromatogram-trace-reconstruction
description: Use when you have an mzML file (e.g. a Beer_multibeers_1_T10_POS.mzML-style
  acquisition) and want to derive the chemical list for a ViMMS simulation from its
  real peaks rather than sampling synthetic chemicals from HMDB.
when_to_use_negative:
- The goal is fully synthetic chemical sampling from databases such as HMDB — use
  ChemicalMixtureCreator instead, since this skill produces unidentified UnknownChemical
  objects from real peaks
- Input data is not an mzML file (e.g. aligned CSV feature tables like multibeers_2_aligned.csv
  are ground-truth files, not ChemicalMixtureFromMZML inputs)
- Identified metabolites with known properties are required — every picked peak is
  treated as a putative chemical with unidentified properties
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Provides ChemicalMixtureFromMZML, the ROI/FeatureExtraction machinery, save_obj
    pickling, and the Environment/controllers that consume the resulting chemical
    list
  repo: https://github.com/glasgowcompbio/vimms
- name: vimms-data
  role: Source of the example mzML data used as input (e.g. Beer_multibeers_1_T10_POS.mzML
    from example_data.zip)
  repo: https://github.com/glasgowcompbio/vimms-data/raw/main/example_data.zip
- name: Python
  role: Runtime for running the extraction and pickling workflow (Python 3.0 or higher)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:5385a686bee894db5fac9c4f528c5db4e16b237ab407c12b3237a6638b9b286d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - vimms-data
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chromatogram-trace-reconstruction

## Summary

Converts an existing experimental mzML file into ViMMS chemical objects by extracting Regions of Interest (ROIs) from its scans via ChemicalMixtureFromMZML, so simulated controllers can be driven by real chromatographic traces instead of synthetic compounds.

## When to use

Use when you have an mzML file (e.g. a Beer_multibeers_1_T10_POS.mzML-style acquisition) and want to derive the chemical list for a ViMMS simulation from its real peaks rather than sampling synthetic chemicals from HMDB.

## When NOT to use

- The goal is fully synthetic chemical sampling from databases such as HMDB — use ChemicalMixtureCreator instead, since this skill produces unidentified UnknownChemical objects from real peaks
- Input data is not an mzML file (e.g. aligned CSV feature tables like multibeers_2_aligned.csv are ground-truth files, not ChemicalMixtureFromMZML inputs)
- Identified metabolites with known properties are required — every picked peak is treated as a putative chemical with unidentified properties

## Inputs

- Experimental mzML file (e.g. Beer_multibeers_1_T10_POS.mzML or mzML output of a prior simulated Top-N run)
- min_intensity threshold parameter controlling which ROIs are retained

## Outputs

- Pickled list of UnknownChemical objects (one per input mzML file), each with m/z, RT and chromatogram attributes populated from the extracted ROIs
- Chemical list consumable by ViMMS controllers and the Environment

## How to apply

Load the source mzML file with ViMMS (e.g. Beer_multibeers_1_T10_POS.mzML, which may be the mzML output of an upstream Top-N run), then instantiate ChemicalMixtureFromMZML, which extracts ROIs from the file's scans via the ROI/FeatureExtraction machinery. Each picked ROI is converted into an UnknownChemical object — every picked peak is treated as a putative chemical with unidentified properties. Call sample() with ms_levels=2 and n_chemicals=None to keep all ROIs, then pickle the resulting list (save_obj) so downstream controllers and the Environment can consume it as an alternative chemical source. Choose the min_intensity threshold deliberately: 1.75E5 completes quickly, 1E4 gives a more accurate but slower simulation, and 0 includes all ROIs including noise.

## Related tools

- **ViMMS 2.0** (Provides ChemicalMixtureFromMZML, the ROI/FeatureExtraction machinery, save_obj pickling, and the Environment/controllers that consume the resulting chemical list) — https://github.com/glasgowcompbio/vimms
- **vimms-data** (Source of the example mzML data used as input (e.g. Beer_multibeers_1_T10_POS.mzML from example_data.zip)) — https://github.com/glasgowcompbio/vimms-data/raw/main/example_data.zip
- **Python** (Runtime for running the extraction and pickling workflow (Python 3.0 or higher))

## Examples

```
from vimms.Chemicals import ChemicalMixtureFromMZML; cm = ChemicalMixtureFromMZML(); chemicals = cm.sample(mzml_file, ms_levels=2, n_chemicals=None); save_obj(chemicals, out_pickle)
```

## Evaluation signals

- The output pickle loads successfully with save_obj/load_obj and its entries are UnknownChemical objects
- Each UnknownChemical has m/z, RT and chromatogram attributes populated from the extracted ROIs
- Calling sample() with n_chemicals=None returns the full set of picked ROIs, whereas a numeric n_chemicals subsamples them
- The chemical list is accepted as input by downstream controllers running inside the Environment
- Varying min_intensity changes the ROI count predictably (1.75E5 → few high-intensity ROIs; 0 → all ROIs including noise)

## Limitations

- min_intensity trade-off: a high threshold like 1.75E5 speeds up the run but discards real peaks, while lowering it to e.g. 1E4 makes simulation runs much slower at higher memory usage
- Setting min_intensity to 0 means using all ROIs as simulation input, including even the noise
- All output chemicals are UnknownChemical objects with unidentified properties — no metabolite identity is assigned

## Evidence

- [abstract] generate chemical objects (produced synthetically or extracted from existing mzML files): "generate chemical objects (produced synthetically or extracted from existing mzML files)"
- [results] ChemicalMixtureFromMZML extracts UnknownChemical objects from mzML: [evidence span withheld — anchor status: not_found]
- [results] min_intensity 1.75E5 set high for speed: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] threshold 0 includes all ROIs including noise: "Setting the threshold to 0 means using all ROIs as simulation input, including even the noise"
- [readme] README: replay previous LC-MS/MS analysis using existing data: [evidence span withheld — anchor status: external_doc]
