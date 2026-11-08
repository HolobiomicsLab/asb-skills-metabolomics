---
name: biomarker-identification-performance-evaluation
description: Use when MS/MS spectra of biomarkers (e.g. from a male infertility study)
  were acquired on an instrument type excluded from DeepMASS training and you must
  test whether the trained scorer still recovers the structure-related metabolite
  from a KEGG reference library.
when_to_use_negative:
- The unknown metabolite has no structure-related metabolite in the reference database
  — the paper states identification is valid only if its structure-related metabolite
  is available in the database
- You need absolute structure elucidation or a direct spectral match to the exact
  compound in a database, rather than structural-similarity-based ranking of candidates
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-learning model that predicts structural similarity between unknown and
    known metabolites from MS/MS spectra and ranks candidate structures
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that acquired the external-test biomarker MS/MS spectra used for
    the unseen-instrument evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:9302cddb22521157a8ab94204d8ea6971176de9fa97a0d121c487dcb079c4b02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# biomarker-identification-performance-evaluation

## Summary

Evaluates DeepMASS-based metabolite identification on external biomarker MS/MS spectra acquired on instruments not used for training (e.g. Shimadzu LC-ESI-IT-TOF), scoring each unknown spectrum against the 662-compound KEGG database and recording whether the structure-related metabolite is the top-ranked hit.

## When to use

Use when MS/MS spectra of biomarkers (e.g. from a male infertility study) were acquired on an instrument type excluded from DeepMASS training and you must test whether the trained scorer still recovers the structure-related metabolite from a KEGG reference library.

## When NOT to use

- The unknown metabolite has no structure-related metabolite in the reference database — the paper states identification is valid only if its structure-related metabolite is available in the database
- You need absolute structure elucidation or a direct spectral match to the exact compound in a database, rather than structural-similarity-based ranking of candidates

## Inputs

- External-test MS/MS spectra of male-infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF (cond_external_shimadzu)
- Trained DeepMASS deep-neural-network structural-similarity scorer (model from hcji/DeepMASS /model/40V directory)
- KEGG reference library of MS/MS spectra for 662 compounds

## Outputs

- external_shimadzu_identification.csv — per-biomarker ranked candidate hits with DeepMASS scores
- Per-biomarker identification-success labels (structure-related metabolite recovered as top hit)
- Reproduced external identification performance (metric_external_shimadzu_identification)

## How to apply

Load the external-test MS/MS spectra of the male-infertility biomarkers measured on the Shimadzu LC-ESI-IT-TOF (cond_external_shimadzu) together with the trained DeepMASS deep-neural-network scorer from the hcji/DeepMASS repository, plus the KEGG reference library of MS/MS spectra for 662 compounds that defines the search space. For each external biomarker spectrum, run DeepMASS inference to score the structural similarity of the unknown metabolite against every KEGG database metabolite, using MS/MS spectra as input to the deep neural network. Rank the database metabolites by DeepMASS score for each spectrum, then label each biomarker as an identification success only when its structure-related (true) metabolite is present in the database AND recovered as the top hit. Assemble the per-biomarker ranked hits and success labels into external_shimadzu_identification.csv and compare against the reported metric_external_shimadzu_identification. The rationale is that structurally related metabolites often have related MS/MS spectra, so identification is valid when a structure-related metabolite is available in the database.

## Related tools

- **DeepMASS** (Deep-learning model that predicts structural similarity between unknown and known metabolites from MS/MS spectra and ranks candidate structures) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument that acquired the external-test biomarker MS/MS spectra used for the unseen-instrument evaluation)

## Examples

```
python example.py  # DeepMASS identification: spectra in data/spectra, trained model unzipped into /model/40V, candidates ranked from the 662-compound KEGG library
```

## Evaluation signals

- external_shimadzu_identification.csv contains, for every biomarker spectrum, a full ranked list of KEGG database metabolites scored by DeepMASS
- Identification is counted as valid only when the structure-related metabolite is present in the database AND recovered as the top-ranked hit — audit the success labels against this rule
- The assembled identification success labels reproduce the reported metric_external_shimadzu_identification
- Search space is exactly the KEGG library of 662 compounds; hits outside this library indicate a pipeline error

## Limitations

- Experimental spectra from the MetDNA dataset were removed from the repository at the data owner's request, so the package cannot be run directly; you must train your own model or wait for public database spectra
- The model and results were validated at 40V collision energy (model-40V); spectra at other energies may not be supported
- External success depends on the structure-related metabolite being present in the KEGG database — unknowns without close structural relatives cannot be identified validly
- Requires a specific dependency stack (RDKit with boost 1.65.1, Keras, IsoSpecPy 1.0.7 under Anaconda Python 3.6)

## Evidence

- [abstract] DeepMASS identification of an unknown compound succeeds when a structure-related metabolite is present in the database, as demonstrated by the external test on male infertility biomarkers measured on the Shimadzu LC-ESI-IT-TOF: [evidence span withheld — anchor status: not_found]
- [abstract] These structurally related metabolites often have related MS/MS spectra, which provide the possibility to identify unknown metabolites through known ones: "These structurally related metabolites often have related MS/MS spectra, which provide the possibility to identify unknown metabolites through known ones"
- [abstract] which can score the structural similarity of unknown metabolite against the known one with MS/MS spectra and deep neural networks: "which can score the structural similarity of unknown metabolite against the known one with MS/MS spectra and deep neural networks"
- [readme] Since the experimental spectra has been removed, this package cannot be run directly.: [evidence span withheld — anchor status: external_doc]
- [abstract] MS/MS spectra of 662 compounds in KEGG: "We evaluated DeepMASS with leave-one-out cross-validation on MS/MS spectra of 662 compounds in KEGG"
