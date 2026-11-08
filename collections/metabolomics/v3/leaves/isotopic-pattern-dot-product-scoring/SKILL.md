---
name: isotopic-pattern-dot-product-scoring
description: Use when you have an aligned peak table (.csv of m/z and retention-time
  pairs) from an HRMS non-targeted analysis and need to annotate features with molecular
  formulae and confirm identifications via isotopic-pattern and MS/MS matching.
when_to_use_negative:
- Features with monoisotopic precursor masses above 500 Da — formula prediction and
  isotopic scoring in MSS explicitly support only small molecules under 500 Da
- Input is raw .mzML data that has not yet been peak-picked and aligned (run mssmain.get_scans
  and mssmain.peak_pick first)
- When you need merged features (isotopes, adducts, in-source fragments combined into
  single chemicals) — feature grouping is not implemented in MSS and is listed as
  future work
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Mass-Suite (MSS)
  role: Hosts the annotation function that queries ChemCalc for candidate formulae,
    computes the dot-product isotopic score, and the frag module for MoNA MS/MS searching
  repo: https://github.com/XiminHu/mass-suite
- name: ChemCalc
  role: Online API used to generate candidate chemical formulae from monoisotopic
    precursor masses
- name: MoNA MassBank database
  role: Online spectral database queried via the MSS fragment module for MS/MS-based
    compound identification
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: pymzML
  role: Parses .mzML raw data into Python-compatible metadata from which observed
    spectra/isotopologue patterns are obtained
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:653257249c9fd8d152b96d8dcf04ce8332a76036fbc1ab3f649cfa3def2bbaca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - ChemCalc
  - Mass-Suite (MSS)
  - MoNA MassBank database
  - Python
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# Isotopic-pattern dot-product scoring

## Summary

A feature-annotation step from the Mass-Suite (MSS) HRMS workflow that assigns candidate chemical formulae to detected features via ChemCalc and ranks them with a dot-product isotopic score comparing theoretical isotopologue patterns to observed spectra, followed by MS/MS confirmation against MoNA/MassBank of North America.

## When to use

Use when you have an aligned peak table (.csv of m/z and retention-time pairs) from an HRMS non-targeted analysis and need to annotate features with molecular formulae and confirm identifications via isotopic-pattern and MS/MS matching.

## When NOT to use

- Features with monoisotopic precursor masses above 500 Da — formula prediction and isotopic scoring in MSS explicitly support only small molecules under 500 Da
- Input is raw .mzML data that has not yet been peak-picked and aligned (run mssmain.get_scans and mssmain.peak_pick first)
- When you need merged features (isotopes, adducts, in-source fragments combined into single chemicals) — feature grouping is not implemented in MSS and is listed as future work

## Inputs

- Aligned feature table as .csv (m/z and retention-time pairs) exported from the MSS peak-detection/alignment stage
- Raw HRMS spectra (from .mzML files parsed with pymzML) for observed isotopologue patterns
- Individual or averaged MS/MS spectra per feature (for MassBank/MoNA fragment matching)

## Outputs

- annotated_features.csv containing formula assignments, dot-product isotopic scores, and MS/MS match results per feature

## How to apply

Load the aligned feature table exported from the MSS peak-detection/alignment stage as a .csv containing m/z and retention-time pairs. For each feature, submit the monoisotopic precursor mass to the ChemCalc online API through the MSS annotation function to generate candidate formulae — note this is supported only for small molecules under 500 Da. Evaluate each candidate formula's prediction accuracy with a dot-product-based isotopic score that compares theoretical isotopologue patterns against the observed spectra, then rank candidates by that score. For compound identification, pass individual or averaged MS/MS spectra through the MSS fragment (frag) module to the MoNA MassBank database, retrieving matches following MassBank database searching criteria and protocols. Finally, write the annotated feature table — formula assignments, isotopic scores, and MS/MS match results — to annotated_features.csv.

## Related tools

- **Mass-Suite (MSS)** (Hosts the annotation function that queries ChemCalc for candidate formulae, computes the dot-product isotopic score, and the frag module for MoNA MS/MS searching) — https://github.com/XiminHu/mass-suite
- **ChemCalc** (Online API used to generate candidate chemical formulae from monoisotopic precursor masses)
- **MoNA MassBank database** (Online spectral database queried via the MSS fragment module for MS/MS-based compound identification) — https://mona.fiehnlab.ucdavis.edu/
- **pymzML** (Parses .mzML raw data into Python-compatible metadata from which observed spectra/isotopologue patterns are obtained) — https://github.com/pymzml/pymzML

## Evaluation signals

- Each annotated feature carries a dot-product isotopic score quantifying agreement between theoretical and observed isotopologue patterns; candidates with low scores should be deprioritized or rejected
- All accepted formula assignments must derive from ChemCalc candidates consistent with the feature's monoisotopic precursor mass (features limited to <500 Da)
- annotated_features.csv must contain, for each feature, a formula assignment, an isotopic score, and MS/MS match results retrieved under MassBank searching criteria
- MS/MS identifications should trace to MoNA MassBank database results, not invented in-house matches
- Features that failed formula prediction or scoring should remain traceable in the output rather than being silently dropped

## Limitations

- Formula prediction and isotopic scoring support only small molecules under 500 Da; larger features cannot be annotated this way
- Feature grouping to merge isotopes, adducts, and in-source fragments into individual chemicals is not implemented in MSS and is listed as future work, so unmerged related features may be scored independently
- Annotation depends on online services (ChemCalc API and MoNA MassBank database); no offline library matching is provided
- Authors caution users to develop a fundamental understanding of the algorithms, their limitations, and assumptions to avoid misinterpretation

## Evidence

- [readme] Formula prediction limited to <500 Da: "2020/09/07: Updated formula prediction and isotopic score function, currently support formula prediction for small molecules under 500 Da"
- [methods] MS/MS confirmation against MassBank of North America: "to the MassBank of North America database [68] for MS/MS fragment matching to facilitate identification"
- [abstract] Annotation as a basic MSS workflow stage: "including both basic functions (e.g., feature extraction, data reduction, feature annotation, data visualization, and statistical analyses)"
- [discussion] Feature grouping not implemented (future work): "e.g., feature grouping to merge the MS features (e.g., isotopes, adducts and in-source fragments) as individual chemicals"
