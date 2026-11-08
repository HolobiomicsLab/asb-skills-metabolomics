---
name: intensity-weighted-spectral-peak-matching
description: 'Use when annotating a tandem MS spectrum by candidate retrieval: you
  have an experimental MS/MS peak list (m/z, intensity) and candidate molecules whose
  theoretical fragmentation patterns must be matched and scored.'
when_to_use_negative:
- The goal is 2D structure rendering or reporting only — RDKit in ChemDistiller is
  used to generate images of 2D chemical structures for the report, not for scoring
- The input is raw LC-MS imaging or profile-mode data rather than a peak-picked tandem
  MS spectrum in ChemDistiller internal format
- No candidate list exists yet — MS1 peak m/z pre-filtering within the --delta_mz
  (ppm) tolerance must be run first to retrieve candidates before fragment matching
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3361
tools:
- name: ChemDistiller
  role: Host engine whose FragScorer/fragprint scorer performs intensity-weighted
    matching of theoretical fragment masses against the normalized experimental MS/MS
    spectrum
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: RDKit
  role: Optional dependency used to parse candidate molecule structure files and enumerate
    bond-breaking fragments for the theoretical peak list
  repo: http://www.rdkit.org/
- name: NumPy
  role: Array operations for peak m/z and normalized intensity vectors in the scoring
    calculation
- name: SciPy
  role: Required numerical library of the ChemDistiller engine running the spectral
    matching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:3ac87b7a50de2425d85bcc3d70ca596aacb20a52136dc0a7e062eae1cea39909
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# intensity-weighted-spectral-peak-matching

## Summary

Scores a candidate metabolite against an experimental tandem MS spectrum by normalizing the spectrum so its peak intensities sum to 1.0 and matching the candidate's pre-computed in silico fragment masses to observed peaks with intensity weighting, as implemented in ChemDistiller's FragScorer (fragprint) scorer.

## When to use

Use when annotating a tandem MS spectrum by candidate retrieval: you have an experimental MS/MS peak list (m/z, intensity) and candidate molecules whose theoretical fragmentation patterns must be matched and scored.

## When NOT to use

- The goal is 2D structure rendering or reporting only — RDKit in ChemDistiller is used to generate images of 2D chemical structures for the report, not for scoring
- The input is raw LC-MS imaging or profile-mode data rather than a peak-picked tandem MS spectrum in ChemDistiller internal format
- No candidate list exists yet — MS1 peak m/z pre-filtering within the --delta_mz (ppm) tolerance must be run first to retrieve candidates before fragment matching

## Inputs

- Experimental tandem MS spectrum as peak m/z and intensity pairs (ChemDistiller internal hierarchical format, e.g. from data/MassBankTestSpectra/*)
- Neutral candidate molecule structure files (SDF/MOL) parsed into molecular graphs
- Chemical compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns

## Outputs

- Intensity-weighted fragment-match score per candidate molecule, suitable for aggregation by a totalscore-style combiner
- Ranked candidate annotations for the MS1 peak (JSON formatted annotations, e.g. annotated_spectra.json)

## How to apply

Load each neutral candidate molecule from a structure file (SDF/MOL) and parse it into a molecular graph with RDKit. Recursively enumerate fragment structures by disconnecting one and then two bonds, applying the ±1 hydrogen tautomerism correction to each fragment formula as done in the ChemDistiller fragprint approach. Compute exact masses of all enumerated fragments to build the theoretical fragment peak list per candidate. Read the experimental tandem MS spectrum (m/z, intensity pairs), normalize its intensities so they sum to 1.0, and score the candidate by matching theoretical fragment masses against the normalized peaks with intensity weighting, in the style of the fragprint scorer. The score is then exposed for aggregation by a totalscore-style combiner alongside other scorers (e.g. FingerScorer SVMs) and the MS1 delta_mz candidate window.

## Related tools

- **ChemDistiller** (Host engine whose FragScorer/fragprint scorer performs intensity-weighted matching of theoretical fragment masses against the normalized experimental MS/MS spectrum) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **RDKit** (Optional dependency used to parse candidate molecule structure files and enumerate bond-breaking fragments for the theoretical peak list) — http://www.rdkit.org/
- **NumPy** (Array operations for peak m/z and normalized intensity vectors in the scoring calculation)
- **SciPy** (Required numerical library of the ChemDistiller engine running the spectral matching)

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- The experimental MS/MS spectrum's normalized intensities sum to 1.0 before matching
- Every theoretical fragment peak derives from one- or two-bond disconnections of the candidate molecular graph with the ±1 hydrogen tautomerism correction applied to fragment formulas
- A smoke-test run on one molecule and one spectrum reproduces a stable score and exposes it for aggregation by a totalscore-style combiner
- Running ChemDistiller in test mode on the 863 MassBank spectra yields retrieval statistics printed at the end of the run
- Matched theoretical fragment m/z values fall within the MS1/fragment m/z tolerance used for candidate search (e.g. --delta_mz in ppm for MS1 peaks)

## Limitations

- Spectra must be in ChemDistiller's internal hierarchical format; format conversions from other formats were not yet implemented at v0.1 release
- RDKit support is optional in the distributed package, and the shipped compound databases (*.h5) and SVM models are small subsets due to size limitations — larger databases must be downloaded separately
- Score quality depends on the pre-computed fragmentation patterns in the database; derivative databases carry licensing caveats from original compound databases for commercial use

## Evidence

- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [intro] high-throughput annotation engine: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
- [methods] msspectra spectrum module: "chemdistiller.msspectra.spectrum module"
- [readme] test mode MassBank spectra: [evidence span withheld — anchor status: external_doc]
- [readme] delta_mz tolerance: [evidence span withheld — anchor status: external_doc]
