---
name: in-silico-msms-fragmentation-enumeration
description: Use when annotating a tandem MS spectrum against candidate molecules
  whose structures are known, and you need pre-calculated fragmentation patterns or
  per-candidate fragment scores to rank metabolite annotations.
when_to_use_negative:
- When candidate molecules lack known structures (no SDF/MOL available), since fragments
  must be enumerated from a molecular graph
- When RDKit is unavailable and only fingerprint/SVM-based scoring (FingerScorer)
  will be used — RDKit is optional in ChemDistiller and mainly used for structure
  images, so graph-based bond breaking is not possible without it
- When the input is already a scored annotation table or feature table rather than
  raw tandem MS peaks requiring fragment matching
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ChemDistiller
  role: Annotation engine whose fragprint/FragScorer design defines the fragment enumeration,
    ±1 hydrogen correction, and intensity-weighted peak matching
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: RDKit
  role: Optional dependency used to parse candidate molecules into molecular graphs
    for bond-breaking fragment enumeration and structure images
- name: NumPy
  role: Required numerical library for fragment mass computation and spectrum/peak
    array handling
- name: SciPy
  role: Required numerical library supporting the scoring and spectrum processing
    routines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:cb7f6676caa80e4f8f7c22d144c4cf90b9b761f17a95710c7b83635ee63a87fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - RDKit
  - SciPy
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# In-silico MS/MS fragmentation enumeration

## Summary

Recursively enumerate fragment structures of a candidate molecule by breaking one and then two bonds with a ±1 hydrogen correction, build a theoretical fragment peak list, and match it against a normalized experimental tandem MS spectrum for candidate scoring as in ChemDistiller's FragScorer.

## When to use

Use when annotating a tandem MS spectrum against candidate molecules whose structures are known, and you need pre-calculated fragmentation patterns or per-candidate fragment scores to rank metabolite annotations.

## When NOT to use

- When candidate molecules lack known structures (no SDF/MOL available), since fragments must be enumerated from a molecular graph
- When RDKit is unavailable and only fingerprint/SVM-based scoring (FingerScorer) will be used — RDKit is optional in ChemDistiller and mainly used for structure images, so graph-based bond breaking is not possible without it
- When the input is already a scored annotation table or feature table rather than raw tandem MS peaks requiring fragment matching

## Inputs

- Neutral candidate molecule structure file (SDF/MOL) parsed with RDKit into a molecular graph
- Experimental tandem MS spectrum as peak m/z and intensity pairs
- MS1 peak m/z value used to restrict candidate compounds within a ppm tolerance (e.g. --delta_mz in ppm)

## Outputs

- Theoretical fragment peak list (exact masses of enumerated fragments) per candidate compound
- Intensity-weighted fragment match score per candidate, exposed for aggregation by a totalscore-style combiner
- Ranked candidate annotations for each tandem MS spectrum

## How to apply

Load the neutral candidate molecule from a structure file (SDF/MOL) and parse it into a molecular graph with RDKit. Recursively enumerate fragments by disconnecting one bond, then two bonds, applying the ±1 hydrogen tautomerism correction to each fragment formula as implemented in the ChemDistiller fragprint approach. Compute exact masses of all enumerated fragments to build a theoretical fragment peak list per candidate. Read the experimental tandem MS spectrum (peak m/z and intensity pairs) and normalize intensities so they sum to 1.0, mirroring chemdistiller.msspectra.spectrum handling. Score each candidate by matching theoretical fragment masses to the normalized experimental peaks with an intensity-weighted match (FragScorer design), exposing the score for aggregation by a totalscore-style combiner that also draws on fingerprint-based scorers such as FingerScorer with Linear_weighted or Radial_weighted SVMs. Package the bond-breaking enumerator and scoring function as a single importable module and validate with a smoke-test on one molecule and one spectrum.

## Related tools

- **ChemDistiller** (Annotation engine whose fragprint/FragScorer design defines the fragment enumeration, ±1 hydrogen correction, and intensity-weighted peak matching) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **RDKit** (Optional dependency used to parse candidate molecules into molecular graphs for bond-breaking fragment enumeration and structure images)
- **NumPy** (Required numerical library for fragment mass computation and spectrum/peak array handling)
- **SciPy** (Required numerical library supporting the scoring and spectrum processing routines)

## Examples

```
python annotate.py Input_Spectra_folder output_folder --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Smoke-test the module on one molecule and one spectrum and confirm the enumerated fragment list is non-empty and all theoretical masses are finite
- Check that experimental MS/MS intensities sum to 1.0 after normalization before matching
- Verify each matched fragment peak respects the spectrum's m/z handling as in chemdistiller.msspectra.spectrum
- Confirm the fragment score is exposed in a form aggregable by the totalscore combiner and comparable against FingerScorer outputs
- Replicate the ChemDistiller --test run (863 MassBank test spectra with known structures) and confirm retrieval statistics are printed

## Limitations

- ChemDistiller benefits from RDKit but it is optional; without it, molecular-graph fragment enumeration and 2D structure images cannot be produced
- Spectra must be in ChemDistiller's internal hierarchical format; conversions from other formats were not yet available in v0.1
- Only one- and two-bond disconnections with a ±1 hydrogen correction are enumerated, so multi-bond rearrangement fragments are not covered
- Default SVMs (Linear_weighted) have worse prediction power than Radial_weighted, which are larger and slower
- Supplied compound databases in HDF5 (*.h5) format are only a small set due to size limitations

## Evidence

- [abstract] Fragment enumeration via recursive bond breaking: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [methods] Spectrum handling module: "chemdistiller.msspectra.spectrum module"
- [intro] Required dependencies: "requires _SciPy_, _NumPy_, _h5py_ libraries."
- [readme] RDKit role: "[RDKit](http://www.rdkit.org/) is currently used to generate
images of 2D chemical structures of compounds for the report"
- [readme] Internal spectrum format constraint: "At the moment your spectra will need to be in __ChemDistiller__ internal format, which is very simple
hierarchical format"
- [readme] Validation via built-in test run: "This will initiate a test annotation run using default settings and the test
data provided in .../ChemDistiller/data/MassBankTestSpectra/*."
