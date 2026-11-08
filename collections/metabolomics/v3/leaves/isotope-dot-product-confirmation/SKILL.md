---
name: isotope-dot-product-confirmation
description: Use when confirming targeted small-molecule identifications from LC-IMS-MS
  raw data where isotope-pattern agreement (idotp), alongside m/z, retention time
  and ion-mobility drift time, is needed to accept or reject a match.
when_to_use_negative:
- Input is already a feature table with curated annotations rather than raw LC-IMS-MS
  data — confirmation against raw isotope envelopes is unnecessary.
- The molecule class has no authentic standards available, so expected isotope envelopes,
  retention times and drift times cannot be defined.
- Data were acquired without ion-mobility separation, when the drift-time dimension
  of the joint-confirmation criterion cannot be evaluated.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Builds the targeted small-molecule method from the conjugated-bile-acid standards
    and re-extracts the raw LC-IMS-MS data, computing isotope dot-product (idotp)
    as part of the joint confirmation criterion
- name: Agilent 6560 IM-Q-ToF MS
  role: Instrument providing the negative-mode LC-IMS-MS raw data (MSV000087889) whose
    drift-time dimension is used in the joint confirmation
- name: GNPS
  role: Provides the orthogonal positive-mode LC-MS/MS library matches against the
    synthetic bile-acid standard library that are merged with the isotope-dot-product-confirmed
    hits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:5293b1c9f694b5b92d9cba11018065d7282b20dfe2394cb878b06326f89fa7ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - GNPS
  - MZmine2
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Isotope dot-product confirmation of small-molecule assignments

## Summary

Confirms targeted small-molecule assignments (e.g. conjugated bile acids) by requiring joint agreement of m/z, retention time, drift time and isotope dot-product (idotp) when re-extracting LC-IMS-MS data in Skyline, as done for Agilent 6560 IM-Q-ToF data of bacterial cultures.

## When to use

Use when confirming targeted small-molecule identifications from LC-IMS-MS raw data where isotope-pattern agreement (idotp), alongside m/z, retention time and ion-mobility drift time, is needed to accept or reject a match.

## When NOT to use

- Input is already a feature table with curated annotations rather than raw LC-IMS-MS data — confirmation against raw isotope envelopes is unnecessary.
- The molecule class has no authentic standards available, so expected isotope envelopes, retention times and drift times cannot be defined.
- Data were acquired without ion-mobility separation, when the drift-time dimension of the joint-confirmation criterion cannot be evaluated.

## Inputs

- Raw LC-IMS-MS data files (e.g. Agilent 6560 IM-Q-ToF, negative mode; MassIVE accession MSV000087889)
- Targeted small-molecule method in Skyline built from synthesized conjugated-bile-acid standards (m/z, retention time, drift time, isotope envelopes)

## Outputs

- Per-compound confirmation status (joint m/z, RT, drift-time and idotp agreement) for each analyte in each sample
- Merged strain-by-conjugate match table (strain_conjugate_match_table.csv) combining LC-IMS-MS confirmation with positive-mode LC-MS/MS library matches and abundances

## How to apply

Build a targeted small-molecule method from authentic standards in Skyline, importing the standards' m/z, retention time, drift time and expected isotope envelopes as extraction parameters. Re-extract the raw LC-IMS-MS data (here, Agilent 6560 IM-Q-ToF, negative mode, MSV000087889 bacterial-culture data) against these parameters, extracting ion chromatograms for each conjugated bile acid conjugate. Accept an assignment only when m/z, retention time and drift time all agree with the standard and the isotope dot-product (idotp) between the observed and expected isotope envelopes is high (joint agreement of all four dimensions). Merge confirmed hits with independent positive-mode LC-MS/MS library matches (e.g. from GNPS searching against the synthetic bile-acid standard library) into a single strain-by-conjugate matrix so that a conjugate is reported for a strain only when both orthogonal confirmations agree.

## Related tools

- **Skyline** (Builds the targeted small-molecule method from the conjugated-bile-acid standards and re-extracts the raw LC-IMS-MS data, computing isotope dot-product (idotp) as part of the joint confirmation criterion)
- **Agilent 6560 IM-Q-ToF MS** (Instrument providing the negative-mode LC-IMS-MS raw data (MSV000087889) whose drift-time dimension is used in the joint confirmation)
- **GNPS** (Provides the orthogonal positive-mode LC-MS/MS library matches against the synthetic bile-acid standard library that are merged with the isotope-dot-product-confirmed hits)

## Examples

```
skyline -e --method bile_acid_targets.sky --input MSV000087889 --filter idotp>0.9
```

## Evaluation signals

- Confirmed compounds show high isotope dot-product (idotp) between observed and expected isotope envelopes, jointly with matching m/z, retention time and drift time.
- A conjugate appears in the strain-by-conjugate matrix only when both the LC-IMS-MS idotp-confirmed extraction and the independent positive-mode LC-MS/MS library match agree.
- The merged output file strain_conjugate_match_table.csv contains one row per strain-conjugate pair with detection/match status and abundance.
- Match counts against known producers (Bifidobacterium, Enterococcus, Clostridium, Cellulosilyticum, Christensenella/Catabacter) are consistent with the published strain screen.

## Limitations

- MS/MS matching and isotope confirmation do not guarantee against false discovery of closely related isomers or compounds; without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded.
- Metabolites must be present in sufficient quantities or ionize well to be detected, so low-abundance conjugates may fail confirmation despite being present.
- The joint-confirmation criterion requires data acquired with ion mobility; instruments lacking drift-time measurement cannot satisfy all four agreement dimensions.

## Evidence

- [results] joint agreement of m/z, retention time, drift time, and isotope dot-product (idotp): [evidence span withheld — anchor status: not_found]
- [results] Build a targeted small-molecule method from the standards in Skyline: [evidence span withheld — anchor status: not_found]
- [supplementary] conjugated bile acids detected in bacterial cultures using an Agilent 6560 IM-Q-ToF MS instrument in negative mode: "conjugated bile acids detected in bacterial cultures using an Agilent 6560 IM-Q-ToF MS instrument in negative mode."
- [discussion] MS/MS matching does not guarantee against false discovery of closely related isomers: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [other] Write the merged table to strain_conjugate_match_table.csv with one row per strain-conjugate pair: [evidence span withheld — anchor status: not_found]
