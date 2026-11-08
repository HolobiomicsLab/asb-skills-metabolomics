---
name: bile-amidate-spectral-library-matching
description: Use when raw positive-mode LC-MS/MS (e.g., Bruker Q-ToF MSV000084475)
  or negative-mode LC-IMS-MS (e.g., Agilent 6560 IM-Q-ToF MSV000087889) culture or
  fecal data must be annotated against synthetic conjugated-bile-acid standard spectra
  to identify per-strain bile amidate producers.
when_to_use_negative:
- The target metabolites are not conjugated bile acids — this skill is restricted
  to bile amidates covered by the synthesized standard library (~2,000 unique compounds
  across three metabolite classes)
- The analyte of interest is present in too low abundance or ionizes poorly and would
  not be selected for MS/MS acquisition, since reverse metabolomics requires an MS/MS
  spectrum
- Only accurate-mass MS1 data are available — the matching protocol requires tandem
  MS spectra (or full LC-IMS-MS features with drift time and isotope pattern)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine2
  role: Feature detection and alignment of the positive-mode Bruker Q-ToF culture
    data before GNPS molecular networking
- name: GNPS
  role: Feature-based molecular networking and library search of culture MS/MS spectra
    against the bile-acid standard library
- name: Skyline
  role: Targeted small-molecule method built from the standards to re-extract the
    Agilent 6560 IM-Q-ToF negative-mode data
- name: MASST
  role: Repository-scale searching of candidate conjugated-bile-acid MS/MS spectra
    in public metabolomics datasets
- name: Bruker Q-ToF
  role: Instrument used to acquire positive-mode MS/MS of conjugated bile acids in
    bacterial cultures (MSV000084475)
- name: Agilent 6560 IM-Q-ToF MS
  role: Instrument used to acquire negative-mode LC-IMS-MS of conjugated bile acids
    in bacterial cultures (MSV000087889)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:9f7b361bce1dab1536640cc20b29fb800c8fde524392a744b9f6630a9bd2c4c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - GNPS
  - MASST
  - MZmine2
  - Skyline
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bile-amidate-spectral-library-matching

## Summary

Matches LC-MS/MS and LC-IMS-MS spectra from bacterial cultures and public metabolomics data against a library of MS/MS spectra synthesized from conjugated bile acid standards (bile amidates) to assign which microbes produce which conjugated bile acids.

## When to use

Use when raw positive-mode LC-MS/MS (e.g., Bruker Q-ToF MSV000084475) or negative-mode LC-IMS-MS (e.g., Agilent 6560 IM-Q-ToF MSV000087889) culture or fecal data must be annotated against synthetic conjugated-bile-acid standard spectra to identify per-strain bile amidate producers.

## When NOT to use

- The target metabolites are not conjugated bile acids — this skill is restricted to bile amidates covered by the synthesized standard library (~2,000 unique compounds across three metabolite classes)
- The analyte of interest is present in too low abundance or ionizes poorly and would not be selected for MS/MS acquisition, since reverse metabolomics requires an MS/MS spectrum
- Only accurate-mass MS1 data are available — the matching protocol requires tandem MS spectra (or full LC-IMS-MS features with drift time and isotope pattern)

## Inputs

- Bacterial-culture positive-mode LC-MS/MS raw data from Bruker Q-ToF (MassIVE MSV000084475)
- Bacterial-culture negative-mode LC-IMS-MS raw data from Agilent 6560 IM-Q-ToF MS (MassIVE MSV000087889)
- MS/MS spectra of synthesized conjugated-bile-acid standards (the reverse metabolomics reference library)

## Outputs

- strain_conjugate_match_table.csv — strain-by-conjugate matrix with detection/match status and abundance for each strain-conjugate pair
- Per-strain feature abundances with amino-acid conjugation assignments from GNPS library search
- Skyline-confirmed conjugate identifications supported by m/z, retention time, drift time, and idotp agreement

## How to apply

Download the bacterial-culture LC-MS/MS raw data (MSV000084475) and LC-IMS-MS raw data (MSV000087889) from MassIVE together with the synthesized conjugated-bile-acid standard spectra used as the reference library. Process the positive-mode Bruker Q-ToF data in MZmine2 for feature detection and alignment, then run GNPS feature-based molecular networking to attach culture MS/MS spectra to each strain's chromatographic features and search them against the bile-acid standard library to assign amino-acid conjugations. Independently, build a targeted small-molecule method from the standards in Skyline and re-extract the Agilent 6560 IM-Q-ToF data in negative mode, requiring joint agreement of m/z, retention time, drift time, and isotope dot-product (idotp) to confirm each conjugate. Merge positive-mode LC-MS/MS and ion-mobility hits into a single strain-by-conjugate matrix of detection/match status and abundance, written to strain_conjugate_match_table.csv with one row per strain-conjugate pair.

## Related tools

- **MZmine2** (Feature detection and alignment of the positive-mode Bruker Q-ToF culture data before GNPS molecular networking)
- **GNPS** (Feature-based molecular networking and library search of culture MS/MS spectra against the bile-acid standard library)
- **Skyline** (Targeted small-molecule method built from the standards to re-extract the Agilent 6560 IM-Q-ToF negative-mode data)
- **MASST** (Repository-scale searching of candidate conjugated-bile-acid MS/MS spectra in public metabolomics datasets)
- **Bruker Q-ToF** (Instrument used to acquire positive-mode MS/MS of conjugated bile acids in bacterial cultures (MSV000084475))
- **Agilent 6560 IM-Q-ToF MS** (Instrument used to acquire negative-mode LC-IMS-MS of conjugated bile acids in bacterial cultures (MSV000087889))

## Examples

```
make run  # run the Nextflow reverse-metabolomics library-generation workflow after conda create -n deploy python=3.8 && pip install -r GNPS2_DeploymentTooling/requirements.txt
```

## Evaluation signals

- strain_conjugate_match_table.csv contains one row per strain-conjugate pair with columns for detection/match status and abundance
- Ion-mobility confirmations require joint agreement of m/z, retention time, drift time, and isotope dot-product (idotp); a call missing any of the four should be flagged unconfirmed
- Producers should concentrate in Gram-positive Bifidobacterium, Enterococcus, Clostridium, Cellulosilyticum and Catabacter (renamed Christensenella) genera; a hit dominated by unrelated genera suggests misannotation
- MASST search results should include USI and MASST job identifiers for all synthesized compounds, enabling audit of every library match
- The MS/MS matching protocol prevents completely inaccurate identifications but not false discoveries of closely related isomers — check isomer-disambiguation status for each assignment

## Limitations

- Without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, although it prevents completely inaccurate identifications
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition, since reverse metabolomics requires an MS/MS spectrum
- The 1.2 billion spectra currently in MASST searches do not encompass all molecules, limiting re-analysis of public data in the discovery phase

## Evidence

- [abstract] Culture of bacteria belonging to the Bifidobacterium , Clostridium and Enterococcus genera produced these bile amidates.: "Culture of bacteria belonging to the Bifidobacterium , Clostridium and Enterococcus genera produced these bile amidates."
- [supplementary] conjugated bile acids detected in bacterial cultures using an Agilent 6560 IM-Q-ToF MS instrument in negative mode.: "conjugated bile acids detected in bacterial cultures using an Agilent 6560 IM-Q-ToF MS instrument in negative mode."
- [supplementary] conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode.: "conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode."
- [discussion] However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data.: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [readme] To learn NextFlow checkout this documentation:: "To run the workflow to test simply do

make run

To learn NextFlow checkout this documentation:"
