---
name: msms-spectral-library-searching
description: Use when LC-MS/MS spectra (mzML) from metabolomics or lipidomics samples
  must be matched against a fragmentation spectral library and identifications ranked
  or scored for precision.
when_to_use_negative:
- Input is LC-MS level-1 (MS-only) metabolomics data without fragmentation spectra
  — MAVEN2's MS/MS matching applies to LC-MS/MS samples
- The spectral library and candidate matches already exist and the goal is de novo
  spectral interpretation rather than library search and scoring
- Only a single scoring method is fixed and no cross-method precision comparison is
  needed; the threshold sweep over five scores would be unnecessary
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MAVEN2
  role: Performs LC-MS/MS spectral matching and efficient search of large-scale fragmentation
    libraries, computing the five match scores used for precision evaluation
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Original open-source LC-MS metabolomics analysis program from which MAVEN2,
    the MS/MS-enabled update, derives
  repo: https://github.com/eugenemel/maven
- name: maven_core
  role: Library dependency used when building MAVEN2 from source for the LC-MS/MS
    search module
  repo: https://github.com/eugenemel/maven_core
- name: CalicoLipidLibrary
  role: Generates in-silico lipid fragmentation spectra as MSP files that can serve
    as the searched spectral library
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:3981dc451b33ea87a1f83181cde1bf760d73058614ad56d161ba44d8e7afbe64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN
  - MAVEN2
  - maven_core
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# msms-spectral-library-searching

## Summary

Search LC-MS/MS spectra (mzML) against a metabolite spectral library in MAVEN2, computing multiple match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) to separate authentic from spurious metabolite identifications.

## When to use

Use when LC-MS/MS spectra (mzML) from metabolomics or lipidomics samples must be matched against a fragmentation spectral library and identifications ranked or scored for precision.

## When NOT to use

- Input is LC-MS level-1 (MS-only) metabolomics data without fragmentation spectra — MAVEN2's MS/MS matching applies to LC-MS/MS samples
- The spectral library and candidate matches already exist and the goal is de novo spectral interpretation rather than library search and scoring
- Only a single scoring method is fixed and no cross-method precision comparison is needed; the threshold sweep over five scores would be unnecessary

## Inputs

- mzML files of LC-MS/MS metabolomics or lipidomics samples (e.g., standards spiked into water and yeast backgrounds)
- A metabolite spectral library (MSP-format fragmentation library, e.g., in-silico lipidomics library)
- A gold-standard set of authentic spiked standards to serve as true positives

## Outputs

- Per-candidate library match scores (dot product, fraction matched, fraction of TIC matched, HGS, MVH) for every identification
- Precision/recall tables per scoring method per threshold and per background (e.g., precision_recall_scores.csv)
- A summary of which scoring method attains the best precision across thresholds

## How to apply

Load mzML files into MAVEN2's LC-MS/MS search module and search each file against a released in-house metabolite spectral library (e.g., an in-silico lipidomics library generated as an MSP file). For every candidate identification, compute the five MAVEN2 match scores: dot product, fraction matched, fraction of TIC matched, highest grouped score (HGS), and MVH. Rank all library matches by each score independently and sweep the score threshold across its range, computing precision (fraction of accepted identifications that are true positives) and recall (fraction of spiked standards recovered) at each threshold. Evaluate separately per sample background (e.g., water vs. yeast) because a more complex background makes discrimination harder; MAVEN2's HGS score is expected to attain the best precision across a large range of thresholds, especially in complex backgrounds.

## Related tools

- **MAVEN2** (Performs LC-MS/MS spectral matching and efficient search of large-scale fragmentation libraries, computing the five match scores used for precision evaluation) — https://github.com/eugenemel/maven
- **MAVEN** (Original open-source LC-MS metabolomics analysis program from which MAVEN2, the MS/MS-enabled update, derives) — https://github.com/eugenemel/maven
- **maven_core** (Library dependency used when building MAVEN2 from source for the LC-MS/MS search module) — https://github.com/eugenemel/maven_core
- **CalicoLipidLibrary** (Generates in-silico lipid fragmentation spectra as MSP files that can serve as the searched spectral library) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
git clone --recursive git@github.com:eugenemel/maven.git maven; cd maven; qmake -r build.pro; make -j4
```

## Evaluation signals

- Output must contain precision and recall values for all five scoring methods at multiple thresholds, for both water and yeast backgrounds
- The comparison should reproduce the paper's reported result that the HGS score yields the best precision across a large range of thresholds, especially in the yeast background
- Precision should be computed as the fraction of accepted identifications that are authentic spiked standards, and recall as the fraction of spiked standards recovered
- Build/run of MAVEN2 from source should succeed (qmake -r build.pro; make -j4) before running the LC-MS/MS search module

## Limitations

- MS/MS library search results depend on library quality; in-silico libraries cover enumerated lipid classes and may omit unsupported classes or adducts (CalicoLipidLibrary errors when fragmentation does not make sense for a chain combination)
- Score-threshold behavior differs by sample background: discrimination is easier in a water background than in a complex yeast background
- Linux builds of Maven have been retired as of 2024-11-05; only macOS and Windows executables are produced by Appveyor
- MVH/HGS and the other scores each rank candidates differently, so no single score is optimal at every threshold — the method comparison is threshold-dependent

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS analysis: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [abstract] Algorithms for MS/MS spectral matching: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] Standards spiked into water and yeast backgrounds: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [readme] README describes generating lipid library MSP file: "# Generate A Lipid Library MSP File

This is the primary usage of this package.  Produce an msp file for all desired in the appropriate ionization mode."
- [readme] README describes building maven from source: "qmake -r build.pro
make -j4"
