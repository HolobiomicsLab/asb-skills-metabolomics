---
name: mass-spectrometry-fragment-annotation
description: Use when tandem MS/MS spectra from a low-resolution miniaturized mass
  spectrometer contain unannotated fragment peaks (e.g. nominal m/z 262 and 249) whose
  molecular formulas or structures must be predicted and cross-validated.
when_to_use_negative:
- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion
  mode (H+) spectra, restricting its applicability
- High-resolution spectra where established tools suffice — the authors note the model
  'may not exhibit exceptional performance advantages' over SIRIUS4, MSNovelist, or
  MassGenie on high-quality resolution spectra
- Compounds outside the element/mass filter space — candidates exceeding 500 Da or
  containing elements beyond C, H, O, N, P, S, Cl, Br, I, F will be excluded by the
  filtering step
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TeFT
  role: Transformer enabled Fragment Tree model that de novo predicts molecular structures
    and formulas from low-resolution MSn spectra
  repo: https://github.com/thumingo/TeFT
- name: RDKit
  role: Mass calculator used to compute monoisotopic masses of TeFT-predicted molecular
    formulas for error verification
- name: pandas
  role: Compiles measured m/z, theoretical m/z, and absolute error per fragment into
    the summary table mass_error_verification.csv
- name: SIRIUS
  role: Baseline mass spectrometry database search method discussed as a comparator
    requiring higher mass resolution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:212f9f8d3caa15f5abc3757e6fae71526a7a5be6026fb8b9d94d3cdee9061d0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - RDKit
  - SIRIUS
  - TeFT
  - pandas
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mass-spectrometry-fragment-annotation

## Summary

Annotate fragmentation peaks of unannotated compounds from low-resolution miniaturized mass spectrometer (MSn) spectra by predicting molecular structures and formulas de novo with the Transformer enabled Fragment Tree (TeFT) model, then verify predicted formulas against high-resolution Orbitrap measurements.

## When to use

Use when tandem MS/MS spectra from a low-resolution miniaturized mass spectrometer contain unannotated fragment peaks (e.g. nominal m/z 262 and 249) whose molecular formulas or structures must be predicted and cross-validated.

## When NOT to use

- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion mode (H+) spectra, restricting its applicability
- High-resolution spectra where established tools suffice — the authors note the model 'may not exhibit exceptional performance advantages' over SIRIUS4, MSNovelist, or MassGenie on high-quality resolution spectra
- Compounds outside the element/mass filter space — candidates exceeding 500 Da or containing elements beyond C, H, O, N, P, S, Cl, Br, I, F will be excluded by the filtering step

## Inputs

- Tandem MS/MS (MSn) spectra from a low-resolution miniaturized linear ion trap mass spectrometer (positive ion mode, H+)
- Peak lists limited to a maximum of 100 peaks selected by ion intensity, with top 5-6 fragmentation peaks and nearby peak points
- High-resolution mass measurements from a Thermo Q-Extractive Orbitrap (Supplementary Fig. S5) for cross-validation

## Outputs

- Predicted molecular structures as SMILES strings generated de novo from spectra
- Predicted molecular formulas with theoretical monoisotopic masses (e.g. C15H24N3O+ at 262.191, C16H13N2O+ at 249.102)
- Mass error verification table (mass_error_verification.csv) with measured m/z, theoretical m/z, and absolute error per fragment

## How to apply

Extract the top five to six fragmentation peaks (limited to at most 100 peaks by ion intensity) from the tandem mass spectrometry spectra, restricting candidate formulas to the ten elements C, H, O, N, P, S, Cl, Br, I, F and molecular weight below 500 Da, with SMILES fragment trees capped at 1500 nodes. Feed these peaks into the TeFT model (a combination of fragmentation tree models and a deep learning Transformer module trained on GNPS, HMDB 5.0, and MoNA in positive ion mode H+) to de novo generate candidate structures and formulas. For validation, compute monoisotopic masses of predicted formulas (e.g. C15H24N3O+ giving 262.191 and C16H13N2O+ giving 249.102) with an RDKit mass calculator, compare against high-resolution Thermo Q-Extractive Orbitrap measurements, and confirm absolute mass errors are below 0.01 Da. Rank candidate substances by similarity between SMILES fragment trees of predicted molecules and fragment trees generated from the tandem MS spectra.

## Related tools

- **TeFT** (Transformer enabled Fragment Tree model that de novo predicts molecular structures and formulas from low-resolution MSn spectra) — https://github.com/thumingo/TeFT
- **RDKit** (Mass calculator used to compute monoisotopic masses of TeFT-predicted molecular formulas for error verification)
- **pandas** (Compiles measured m/z, theoretical m/z, and absolute error per fragment into the summary table mass_error_verification.csv)
- **SIRIUS** (Baseline mass spectrometry database search method discussed as a comparator requiring higher mass resolution)

## Examples

```
python verify_mass_errors.py --theoretical C15H24N3O+ C16H13N2O+ --measured 262.191,249.111 --tolerance 0.01 --out mass_error_verification.csv
```

## Evaluation signals

- Absolute mass error between TeFT-predicted formula monoisotopic mass and Orbitrap measured m/z is below 0.01 Da for each fragment (e.g. m/z 262: 262.191 vs 262.191)
- Predicted formula for the nominal 262 peak matches C15H24N3O+ and for 249 matches C16H13N2O+ against Orbitrap measurements
- Ranked candidate structures show similarity between SMILES fragment trees of predictions and fragment trees built from tandem MS spectra
- Output table mass_error_verification.csv contains columns for measured m/z, theoretical m/z, and absolute error for every validated fragment
- For flavonoid alcohol benchmarks on the miniaturized instrument, complete structure prediction is achieved for a majority-of-test-standard proportion (8 of 16 substances)

## Limitations

- TeFT was trained only on spectra recorded in positive ion mode (H+), restricting applicability to negative ion mode data
- Applied best to fragmentation peaks with clear fragmentation rules; performance on spectra without such rules is untested
- Model may not outperform high-resolution-focused deep models (SIRIUS4, MSNovelist, MassGenie) on high-quality resolution spectra
- Training-scale constraints of miniaturized on-site platforms prevent large-scale model training similar to MassGenie
- Error correction relies on multiple measurement averaging and deconvolution integration of spectrogram data; single-pass spectra may be noisier

## Evidence

- [discussion] TeFT is a de novo structure predictor for low-resolution MSn: "We propose a novel prediction model, TeFT, designed for de novo structure generation from low-resolution tandem mass spectrometry (MSn) spectra"
- [discussion] Formula filter constraints: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] Element restriction filter: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] Peak selection scheme: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Ranking via fragment-tree similarity: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [abstract] Confirmed annotation of Anweiyang capsule compound: "Finally, TeFT confirmed the structure of the compound contained in a Chinese medicine substance called the Anweiyang capsule"
- [discussion] Positive-ion-only training limitation: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
- [abstract] Flavonoid benchmark result: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [readme] README description of TeFT purpose: "It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra"
