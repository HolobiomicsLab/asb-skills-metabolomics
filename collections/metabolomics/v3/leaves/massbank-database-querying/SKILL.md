---
name: massbank-database-querying
description: Use when an aligned HRMS feature table (m/z + retention-time pairs) exists
  and features must be assigned candidate formulae and compound identities via MS/MS
  spectral matching against MassBank/MoNA.
when_to_use_negative:
- Input is still raw .mzML data that has not been peak-picked and aligned (run mssmain.peak_pick
  and alignment first)
- Features correspond to compounds above 500 Da, where MSS formula prediction is not
  supported
- The goal is feature clustering, dilution prediction, or source tracking — use the
  dm.ms_cluster / feature_model analysis module instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Mass-Suite (MSS)
  role: Provides the fragment module (mss/frag.py) that uploads MS/MS spectra to the
    MoNA MassBank endpoint and the annotation function for ChemCalc formula prediction
    and isotopic scoring
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Parses raw .mzML files into Python-compatible metadata so MS/MS spectra can
    be extracted for downstream MassBank querying
  repo: https://github.com/pymzml/pymzML
- name: MoNA MassBank database
  role: Online spectral database searched by the MSS fragment module using MassBank
    database searching criteria and protocols
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:7893538aa5f750d92735acd9f515ea35af984a0ba1996277518b3e6275b17133
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - MassBank of North America
  - MoNA MassBank database
  - Python
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# massbank-database-querying

## Summary

Annotates HRMS features from the MSS pipeline by querying the MassBank of North America (MoNA endpoint) with individual or averaged MS/MS spectra, complemented by ChemCalc formula prediction and dot-product isotopic scoring, to produce a fully annotated feature table.

## When to use

Use when an aligned HRMS feature table (m/z + retention-time pairs) exists and features must be assigned candidate formulae and compound identities via MS/MS spectral matching against MassBank/MoNA.

## When NOT to use

- Input is still raw .mzML data that has not been peak-picked and aligned (run mssmain.peak_pick and alignment first)
- Features correspond to compounds above 500 Da, where MSS formula prediction is not supported
- The goal is feature clustering, dilution prediction, or source tracking — use the dm.ms_cluster / feature_model analysis module instead

## Inputs

- Aligned feature table (.csv) containing m/z and retention-time pairs from the upstream peak-detection/alignment stage
- Individual or averaged MS/MS spectra exported from the MSS fragment module

## Outputs

- annotated_features.csv including formula assignments, isotopic scores, and MS/MS match results

## How to apply

Load the aligned feature table exported from the upstream peak-detection/alignment stage as a .csv containing m/z and retention-time pairs. For each feature, submit its monoisotopic precursor mass to the ChemCalc online API via the MSS annotation function to generate candidate formulae (supported for small molecules under 500 Da), then score each candidate with a dot-product-based isotopic score comparing theoretical isotopologue patterns against observed spectra. For compound identification, upload individual or averaged MS/MS spectra through the MSS fragment module (mss/frag.py) to the MoNA MassBank database endpoint, following MassBank database searching criteria and protocols. Finally, write formula assignments, isotopic scores, and MS/MS match results to annotated_features.csv.

## Related tools

- **Mass-Suite (MSS)** (Provides the fragment module (mss/frag.py) that uploads MS/MS spectra to the MoNA MassBank endpoint and the annotation function for ChemCalc formula prediction and isotopic scoring) — https://github.com/XiminHu/mass-suite
- **pymzML** (Parses raw .mzML files into Python-compatible metadata so MS/MS spectra can be extracted for downstream MassBank querying) — https://github.com/pymzml/pymzML
- **MoNA MassBank database** (Online spectral database searched by the MSS fragment module using MassBank database searching criteria and protocols)

## Evaluation signals

- annotated_features.csv is produced and contains, for each feature, formula assignments, isotopic scores, and MS/MS match results
- Each submitted feature's monoisotopic precursor mass yields ChemCalc candidate formulae with a dot-product-based isotopic score against observed isotopologue patterns
- MS/MS matches are returned from the MoNA endpoint following MassBank searching criteria (spectra submitted as individual or averaged fragments)
- Formula predictions are only expected to succeed for small molecules under 500 Da — check that failure cases fall in this known limitation rather than a pipeline error

## Limitations

- Formula prediction supports only small molecules under 500 Da
- Identification depends on online availability and coverage of the MoNA MassBank database; matches follow MassBank database searching criteria and protocols
- Feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented and is listed as future work, so annotation operates per-feature
- Authors strongly encourage users to develop understanding of the algorithms, their limitations and assumptions, to avoid misinterpretation of results

## Evidence

- [methods] Upload individual or averaged MS/MS spectra through the MSS fragment module to the MassBank of North America database (MoNA endpoint): "to the MassBank of North America database [68] for MS/MS fragment matching to facilitate identification"
- [readme] Formula prediction is limited to small molecules under 500 Da and scored by dot-product isotopic comparison: "Updated formula prediction and isotopic score function, currently support formula prediction for small molecules under 500 Da"
- [readme] All MSS online spectral searching targets the MoNA MassBank database: "All the online searching is using [MoNA MassBank database](https://mona.fiehnlab.ucdavis.edu/)."
- [discussion] Users must understand algorithm limitations and assumptions to avoid misinterpreting annotation results: "we do strongly encourage users to develop fundamental understanding about the algorithms, and their limitations and assumptions"
