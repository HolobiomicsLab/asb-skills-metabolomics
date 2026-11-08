---
name: mgf-spectrum-parsing
description: Use when tandem MS/MS data are provided as MGF peak lists (e.g. from
  a ProteomeXchange/PRIDE deposit such as PXD034932) and spectra must be read with
  precursor m/z for direct spectral comparison or parameter estimation.
when_to_use_negative:
- Input is raw instrument data (e.g. vendor RAW or mzML) rather than MGF peak lists
  — convert to peak lists first, since compareMS2 operates on MGF spectra
- The goal is peptide/protein identification via database search; compareMS2 works
  before and independently of database search and does not assign spectra to peptide
  sequences
- Only a similarity label between two datasets is needed with no spectral detail —
  use compareMS2's phylogenetic tree or spectra2species modes directly instead of
  custom parsing
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: compareMS2
  role: Tool whose R scripting examples parse MGF files and define the eq 1 spectral
    similarity used on the parsed spectra; downstream parameters (m/z windows, similarity
    thresholds) are tuned from the parsed pairs
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:cda185b7973b47dd4f20414e28c59461ea9227521b75d577e4662ac2d8384c7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mgf-spectrum-parsing

## Summary

Parse MGF (Mascot Generic Format) peak list files so that each tandem mass spectrum is read together with its precursor m/z, enabling direct spectrum-to-spectrum comparison in compareMS2-style analyses before any database search.

## When to use

Use when tandem MS/MS data are provided as MGF peak lists (e.g. from a ProteomeXchange/PRIDE deposit such as PXD034932) and spectra must be read with precursor m/z for direct spectral comparison or parameter estimation.

## When NOT to use

- Input is raw instrument data (e.g. vendor RAW or mzML) rather than MGF peak lists — convert to peak lists first, since compareMS2 operates on MGF spectra
- The goal is peptide/protein identification via database search; compareMS2 works before and independently of database search and does not assign spectra to peptide sequences
- Only a similarity label between two datasets is needed with no spectral detail — use compareMS2's phylogenetic tree or spectra2species modes directly instead of custom parsing

## Inputs

- MGF peak list files (one per LC-MS/MS dataset, each tandem mass spectrum with its precursor m/z)
- R scripting examples for MGF parsing and spectral similarity calculation (osf.io/jey28)

## Outputs

- Parsed spectra: lists of fragment m/z and intensity per spectrum, keyed by precursor m/z
- Spectrum-pair similarity values (eq 1 cosine-style overlap) per precursor m/z window
- Optionally a 2D histogram of precursor m/z difference versus spectral similarity (mzdiff_similarity_histogram.png)

## How to apply

Obtain the MGF peak list files for the LC-MS/MS datasets of interest (each of the five serum datasets in PXD034932 contains between 42,629 and 47,626 tandem mass spectra) and the R scripting examples distributed via the compareMS2 project (osf.io/jey28). Parse each MGF entry into a tandem mass spectrum with its precursor m/z, then compute the spectral similarity of eq 1 (a cosine-style overlap of the two spectra) for spectrum pairs within a precursor m/z window. Because compareMS2 compares sets of tandem mass spectra directly with each other — rather than against predicted spectra of specific peptides — no database search, spectral library, or FDR estimator such as PeptideProphet or Percolator is required at this stage. Binning parsed spectrum pairs by precursor m/z difference and similarity supports estimating suitable compareMS2 parameters such as m/z windows and spectral similarity thresholds before running the tool's phylogenetic-tree, spectra2species, or heatmap modes.

## Related tools

- **compareMS2** (Tool whose R scripting examples parse MGF files and define the eq 1 spectral similarity used on the parsed spectra; downstream parameters (m/z windows, similarity thresholds) are tuned from the parsed pairs) — https://github.com/524D/compareMS2.git

## Evaluation signals

- Parsed spectrum counts per dataset fall in the range reported for the serum datasets (42,629–47,626 tandem mass spectra each)
- Every parsed spectrum carries a precursor m/z, since the x-axis of the diagnostic histogram is precursor m/z difference
- Identical spectral content gives similarity 1 and distance 0; dissimilar dataset pairs trend toward similarity 0
- The 2D histogram of precursor m/z difference vs. similarity reproduces the paper's pattern (human vs. chimpanzee shows high similarity at low m/z difference; human vs. E. coli shows few similar tryptic peptide spectra) and reveals charge states, mass measurement precision and isotope errors
- Estimated m/z windows and spectral similarity thresholds are reasonable for running compareMS2 pairwise comparisons

## Limitations

- The workflow depends on external MGF test data and R scripting examples (ProteomeXchange PXD034932, osf.io/jey28), not on code shipped in the main README
- The Jaccard similarity variant is very sensitive to the precise definition of when two spectra are considered identical
- Results depend on protein abundance/coverage, not only peptide sequence, when comparing proteomes across species or tissues

## Evidence

- [results] similarity as function of m/z difference: "we compared the similarity between tandem mass spectra as a function of precursor m/z difference"
- [results] parameter estimation: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [results] eq 1 overlap: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [readme] README scope: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] README test data: [evidence span withheld — anchor status: not_found]
