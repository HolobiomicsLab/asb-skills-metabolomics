---
name: bond-disconnection-fragment-pattern-generation
description: Use when candidate compound structures (SDF/MOL) must be converted into
  theoretical MS/MS fragment peak lists by recursive 1- and 2-bond disconnection,
  then scored against a normalized experimental tandem MS spectrum for metabolite
  annotation retrieval.
when_to_use_negative:
- Candidate retrieval should be driven by molecular fingerprint similarity (FingerScorer
  with SVM models) rather than fragment-peak matching
- The goal is running the full annotation pipeline over a spectrum folder rather than
  scoring individual candidate molecules
- No molecular structure is available for the candidate, so a molecular graph cannot
  be built and fragments cannot be enumerated
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3473
tools:
- name: ChemDistiller
  role: Source of the fragprint bond-breaking fragmentation design, ±1 hydrogen tautomerism
    correction, and spectrum normalization conventions mirrored by this skill
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: RDKit
  role: Parses SDF/MOL candidate structures into molecular graphs used for recursive
    bond disconnection and fragment enumeration
  repo: http://www.rdkit.org/
- name: NumPy
  role: Numerical computation of exact fragment masses and intensity-weighted peak
    matching arrays
- name: SciPy
  role: Supporting numerical library for the scoring computation, listed as a ChemDistiller
    dependency
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:67ba8e2bd655be992ff92a7666d3d9bb4cefa7001a4a7cb9c9d3723bb29e6e58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - RDKit
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# bond-disconnection-fragment-pattern-generation

## Summary

Reconstructs the FragScorer-style in silico fragmentation of ChemDistiller: recursively breaking one and then two bonds of a candidate molecule (with ±1 hydrogen tautomer correction) to build theoretical fragment peak lists that are matched against an intensity-normalized experimental tandem MS spectrum.

## When to use

Use when candidate compound structures (SDF/MOL) must be converted into theoretical MS/MS fragment peak lists by recursive 1- and 2-bond disconnection, then scored against a normalized experimental tandem MS spectrum for metabolite annotation retrieval.

## When NOT to use

- Candidate retrieval should be driven by molecular fingerprint similarity (FingerScorer with SVM models) rather than fragment-peak matching
- The goal is running the full annotation pipeline over a spectrum folder rather than scoring individual candidate molecules
- No molecular structure is available for the candidate, so a molecular graph cannot be built and fragments cannot be enumerated

## Inputs

- Neutral candidate molecule structure file (SDF/MOL) parsed into a molecular graph via RDKit
- Experimental tandem MS spectrum as peak m/z and intensity pairs (ChemDistiller internal hierarchical spectrum format)

## Outputs

- Theoretical fragment peak list (exact masses of enumerated 1- and 2-bond disconnection fragments) per candidate compound
- Fragment formulas with ±1 hydrogen tautomerism correction
- Intensity-weighted fragment-match score per candidate, suitable for aggregation by a totalscore-style combiner

## How to apply

Load the neutral candidate molecule from an SDF/MOL structure file and parse it into a molecular graph with RDKit. Recursively enumerate fragments by disconnecting one and then two bonds per molecule, applying the ±1 hydrogen tautomerism correction to each fragment formula as in the ChemDistiller fragprint approach. Compute exact masses of all enumerated fragments to build a theoretical fragment peak list per candidate. Read the experimental tandem MS spectrum (m/z, intensity pairs), normalize so intensities sum to 1.0, and score each candidate by intensity-weighted matching of theoretical fragment masses to normalized peaks, exposing the score for aggregation by a totalscore-style combiner. Validate by running the packaged module on one molecule and one spectrum as a smoke test.

## Related tools

- **ChemDistiller** (Source of the fragprint bond-breaking fragmentation design, ±1 hydrogen tautomerism correction, and spectrum normalization conventions mirrored by this skill) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **RDKit** (Parses SDF/MOL candidate structures into molecular graphs used for recursive bond disconnection and fragment enumeration) — http://www.rdkit.org/
- **NumPy** (Numerical computation of exact fragment masses and intensity-weighted peak matching arrays)
- **SciPy** (Supporting numerical library for the scoring computation, listed as a ChemDistiller dependency)

## Examples

```
python annotate.py --test
```

## Evaluation signals

- Smoke-test script on one molecule and one spectrum runs end-to-end and produces a finite score
- Every enumerated fragment formula equals the parent formula minus a bond disconnection with the ±1 hydrogen correction applied
- Experimental peak intensities sum to exactly 1.0 after normalization before matching
- Theoretical fragment masses are matched against normalized experimental peaks within the instrument m/z tolerance (e.g. the --delta_mz ppm tolerance used by ChemDistiller)

## Limitations

- Fragment enumeration is limited to disconnecting one and then two bonds per molecule; deeper fragmentation is not enumerated
- Spectra must be supplied in ChemDistiller's internal hierarchical format; conversions from other formats were not yet implemented at the time of the README
- RDKit is optional in ChemDistiller overall (used for 2D structure images in reports), so environments without RDKit need alternative structure parsing

## Evidence

- [abstract] Pre-calculated database fragmentation underpins candidate retrieval: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [intro] ChemDistiller is a tandem MS annotation engine: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
- [readme] RDKit is the structure-parsing dependency: "__ChemDistiller__ benefits from installed and available [RDKit](http://www.rdkit.org/), however it is optional"
- [readme] Smoke-test validation path with MassBank test spectra: [evidence span withheld — anchor status: external_doc]
- [methods] Spectra handled via the msspectra.spectrum module: "chemdistiller.msspectra.spectrum module
---------------------------------------

.. automodule:: chemdistiller.msspectra.spectrum"
