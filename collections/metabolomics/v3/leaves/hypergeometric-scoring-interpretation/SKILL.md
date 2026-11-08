---
name: hypergeometric-scoring-interpretation
description: Use when MAVEN2 lipidomics MS/MS search results must be annotated and
  scored, i.e., per-feature lipid annotations from two in-silico libraries (CalicoLipids,
  MS-DIAL) need quality assessment via the hypergeometric score before downstream
  thresholding or counting.
when_to_use_negative:
- Input is already a scored/annotated feature table from both libraries — no further
  MAVEN2 search or scoring is needed.
- Data are metabolomics-only without MS/MS fragmentation spectra — the hypergeometric
  score applies to MS/MS spectral matching, not MS1-only peak annotation.
- Only one library search exists and no cross-library comparison (CalicoLipids vs
  MS-DIAL) is intended — a single-library annotation without dual-library context
  is out of scope for this skill.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: MAVEN2
  role: Performs the MS/MS spectral matching search of lipidomics features against
    in-silico libraries and computes per-feature hypergeometric scores and annotations
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: GUI used to load mzML search samples and reference fragmentation libraries,
    and run the MS/MS search interactively
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation MSP library (CalicoLipids) used
    as one reference library in the dual-library search
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:49deb1cd7ab9c6718101f1f6992445969c45a603087f257f356fac7e300dd5c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hypergeometric-scoring-interpretation

## Summary

Interpret MAVEN2 MS/MS spectral-matching results for lipidomics features, where the hypergeometric score is the metric of spectral match quality and annotations are compared across the CalicoLipids and MS-DIAL in-silico libraries rather than trusted from a single library.

## When to use

Use when MAVEN2 lipidomics MS/MS search results must be annotated and scored, i.e., per-feature lipid annotations from two in-silico libraries (CalicoLipids, MS-DIAL) need quality assessment via the hypergeometric score before downstream thresholding or counting.

## When NOT to use

- Input is already a scored/annotated feature table from both libraries — no further MAVEN2 search or scoring is needed.
- Data are metabolomics-only without MS/MS fragmentation spectra — the hypergeometric score applies to MS/MS spectral matching, not MS1-only peak annotation.
- Only one library search exists and no cross-library comparison (CalicoLipids vs MS-DIAL) is intended — a single-library annotation without dual-library context is out of scope for this skill.

## Inputs

- In-house Thermo lipidomics mzML files (search samples)
- CalicoLipids in-silico lipid fragmentation library (MSP file generated from calico/CalicoLipidLibrary)
- MS-DIAL in-silico lipid library (alternate reference library)
- MAVEN2 GUI search configuration for MS/MS spectral matching

## Outputs

- CSV table of per-feature hypergeometric scores and best lipid annotations from the CalicoLipids search
- CSV table of per-feature hypergeometric scores and best lipid annotations from the MS-DIAL search
- Per-feature proposed annotations paired with spectral match scores for downstream thresholding and counting

## How to apply

Load the Thermo lipidomics mzML files into MAVEN2 as search samples and run MS/MS spectral matching against an in-silico lipid fragmentation library loaded as the reference (e.g., an MSP file generated from the calico/CalicoLipidLibrary repository). MAVEN2 computes a hypergeometric score per feature that captures how well the observed spectrum matches the theoretical library spectrum, alongside the best proposed lipid annotation. Run the identical search twice — once against the CalicoLipids library and once against the MS-DIAL library — and export each result as a structured CSV containing feature identifiers, hypergeometric scores, and annotations. The rationale is that a novel in-silico lipidomics library (CalicoLipids) is compared against existing in-silico libraries (MS-DIAL) rather than a single library being taken as ground truth, and MAVEN2's approach to separating authentic from spurious identifications was validated using standards spiked into water and yeast backgrounds. Retain both per-library score/annotation tables so downstream steps can threshold and count consistent cross-library annotations.

## Related tools

- **MAVEN2** (Performs the MS/MS spectral matching search of lipidomics features against in-silico libraries and computes per-feature hypergeometric scores and annotations) — https://github.com/eugenemel/maven
- **Maven GUI** (GUI used to load mzML search samples and reference fragmentation libraries, and run the MS/MS search interactively) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation MSP library (CalicoLipids) used as one reference library in the dual-library search) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -n "calicolipids_pos.msp"
```

## Evaluation signals

- Each exported CSV table contains a hypergeometric score entry for every searched feature.
- Each exported CSV table contains a lipid annotation entry for every searched feature.
- Two separate structured CSV outputs exist, one per reference library (CalicoLipids and MS-DIAL).
- Both tables parse as valid CSV and retain feature identifiers, scores, and annotations.
- Annotation proposals from the two libraries can be compared feature-by-feature for downstream thresholding and counting analyses.

## Limitations

- In-silico library searches can produce spurious identifications; MAVEN2's scoring must separate authentic from spurious matches, which was assessed only with standards spiked into water and yeast backgrounds.
- Enumerating all theoretical spectra over all lipid classes with CalicoLipidLibrary may be time-consuming; the -c option should be used to restrict generation to classes of interest.
- Not all lipid classes/adduct combinations are supported by CalicoLipidLibrary — unsupported class/adduct requests produce an error or a list of supported options.
- Changes for some MAVEN2 releases (e.g., 8.0.0) are largely undocumented, which may complicate reproducing exact search behavior.

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS spectral matching and efficient search of large-scale fragmentation libraries: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] Novel in-silico lipidomics library compared against existing libraries: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [abstract] Separating authentic from spurious identifications validated with spiked standards: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [readme] CalicoLipidLibrary generates in-silico lipid fragmentation spectra for the reference MSP library: [evidence span withheld — anchor status: external_doc]
- [abstract] MAVEN2 is the updated open-source platform supporting metabolomics and lipidomics LC-MS/MS analysis: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
