---
name: in-silico-lipid-library-search
description: Use when per-feature lipid annotations are needed for Thermo mzML lipidomics
  data via MS/MS spectral matching in MAVEN2 against the CalicoLipids and MS-DIAL
  in-silico libraries scored by hypergeometric score.
when_to_use_negative:
- Input is already a completed feature annotation/quantification table — no MS/MS
  library search is needed
- Data are unlabeled LC-MS metabolomics without MS/MS spectra, since the skill depends
  on MS/MS spectral matching against a fragmentation library
- The task is library generation itself (building msp files with CalicoLipidLibrary)
  rather than searching samples against a built library
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Runs the MS/MS spectral matching search of detected features against the in-silico
    lipid libraries and computes hypergeometric match scores
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: GUI used to load Thermo mzML samples and reference libraries and run the per-library
    searches
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the CalicoLipids in-silico msp lipid fragmentation library used
    as one of the two reference libraries
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:10fa1cba03a6e879bfcf24843ba050a35194be16963c95e0e32d20569e7d5c0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  - git
  - make
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# in-silico-lipid-library-search

## Summary

Search LC-MS/MS lipidomics features in MAVEN2 against in-silico lipid fragmentation libraries (CalicoLipids and MS-DIAL) using MS/MS spectral matching with the hypergeometric score to assign per-feature lipid annotations and match-quality metrics.

## When to use

Use when per-feature lipid annotations are needed for Thermo mzML lipidomics data via MS/MS spectral matching in MAVEN2 against the CalicoLipids and MS-DIAL in-silico libraries scored by hypergeometric score.

## When NOT to use

- Input is already a completed feature annotation/quantification table — no MS/MS library search is needed
- Data are unlabeled LC-MS metabolomics without MS/MS spectra, since the skill depends on MS/MS spectral matching against a fragmentation library
- The task is library generation itself (building msp files with CalicoLipidLibrary) rather than searching samples against a built library

## Inputs

- Thermo lipidomics mzML files (LC-MS/MS samples)
- CalicoLipids in-silico lipid spectral library (msp-format, from calico/CalicoLipidLibrary / MTBLS3097)
- MS-DIAL in-silico lipid library (msp-format)
- Detected features requiring MS/MS annotation

## Outputs

- Per-feature CSV annotation table from the CalicoLipids library search (feature IDs, hypergeometric scores, lipid annotations)
- Per-feature CSV annotation table from the MS-DIAL library search (feature IDs, hypergeometric scores, lipid annotations)

## How to apply

Load the in-house Thermo mzML lipidomics files into the MAVEN2 GUI as search samples, then load the CalicoLipids in-silico library (generated from calico/CalicoLipidLibrary, deposited as MTBLS3097) as the reference fragmentation library and run the MAVEN2 MS/MS spectral matching search, recording each feature's hypergeometric score and best lipid annotation. Load the MS-DIAL in-silico lipid library as the alternate reference and re-run the identical search, again capturing per-feature hypergeometric scores and proposed annotations. Export the two search outputs as structured CSV tables (one per library) retaining feature identifiers, scores, and annotations for downstream thresholding and counting. Validate that each CSV parses, covers every searched feature, and contains both a hypergeometric score and a lipid annotation entry from both library searches. Rationale: MAVEN2's algorithms support MS/MS spectral matching and efficient search of large-scale fragmentation libraries, enabling direct comparison of annotation quality between the novel CalicoLipids library and an existing in-silico library.

## Related tools

- **MAVEN2** (Runs the MS/MS spectral matching search of detected features against the in-silico lipid libraries and computes hypergeometric match scores) — https://github.com/eugenemel/maven
- **Maven GUI** (GUI used to load Thermo mzML samples and reference libraries and run the per-library searches) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the CalicoLipids in-silico msp lipid fragmentation library used as one of the two reference libraries) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -o ./lib -n lipids_pos.msp
```

## Evaluation signals

- Each exported CSV parses as valid CSV and contains a hypergeometric score and a lipid annotation entry for every searched feature
- Two separate output tables exist, one per reference library (CalicoLipids and MS-DIAL), with matching feature identifiers
- Sampled per-feature annotations can be cross-checked against the MTBLS3097 reference deposit for library provenance
- The identical search configuration (samples, scoring) is re-used across both library runs so scores are comparable
- Authentic vs. spurious identifications separate as expected, as benchmarked with standards spiked into water and yeast backgrounds

## Limitations

- In-silico libraries are rule-based approximations; fragment coverage depends on the generation rules (61 lipid classes, 15 adduct forms, HCD-mimicking stepped NCE 20/30/40) rather than measured spectra
- Not all adduct forms are valid for all lipid classes in the CalicoLipidLibrary enumeration
- MAVEN2 library search performance against very large fragmentation libraries is a stated algorithmic focus; extremely large custom libraries may still be costly to search
- Changelog gaps: changes for releases 8.0.0 and 20161028 are largely undocumented, complicating reproducibility of exact search behavior across versions

## Evidence

- [results] All lipidomics samples were searched in MAVEN2 against both the CalicoLipids spectral library and the MS-DIAL spectral library, and the hypergeometric score was used to assess spectral quality: [evidence span withheld — anchor status: not_found]
- [abstract] algorithms to support MS/MS spectral matching: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [readme] README usage of generateDB.py: [evidence span withheld — anchor status: external_doc]
- [intro] GUI identity: "# Maven GUI: Metabolomics Analysis and Visualization Engine"
