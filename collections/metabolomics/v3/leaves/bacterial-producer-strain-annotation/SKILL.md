---
name: bacterial-producer-strain-annotation
description: Use when screening bacterial culture MS data (e.g. HMP isolates, MSV000084475
  and MSV000087889) to determine which genera or strains produced specific amino-acid-conjugated
  bile acids, requiring joint agreement of m/z, retention time, drift time, and isotope
  dot-product.
when_to_use_negative:
- When the input is human stool or IBD fecal sample data rather than bacterial-culture
  isolates — the workflow is specific to strain-level culture screening
- When no synthesized conjugated-bile-acid standard spectra are available as a reference
  library, since the skill depends on searching culture spectra against those standards
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0621
tools:
- name: MZmine2
  role: Feature detection and alignment on the positive-mode LC-MS/MS bacterial-culture
    data before GNPS molecular networking
- name: GNPS
  role: Feature-based molecular networking and searching culture MS/MS spectra against
    the bile-acid standard library to assign conjugations and extract per-strain abundances
- name: Skyline
  role: Targeted small-molecule method built from the synthesized standards to re-extract
    the LC-IMS-MS data
- name: Agilent 6560 IM-Q-ToF MS
  role: Instrument generating negative-mode ion-mobility MS data used for confirmation
    of conjugates via drift time and isotope dot-product
- name: Bruker Q-ToF
  role: Instrument generating positive-mode LC-MS/MS data for the bacterial-culture
    screen
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:5f62398c3fe169f2eb40e3fbb991c392665568ed8bc0f8df2694888dd600735e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - GNPS
  - MZmine2
  - Skyline
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bacterial-producer-strain-annotation

## Summary

Identifies which bacterial isolates produced which conjugated bile acids by matching each strain's LC-MS/MS and ion-mobility spectra against a library of synthesized conjugated-bile-acid standards, merging positive-mode LC-MS/MS (Bruker Q-ToF) and negative-mode LC-IMS-MS (Agilent 6560) hits into a strain-by-conjugate producer matrix.

## When to use

Use when screening bacterial culture MS data (e.g. HMP isolates, MSV000084475 and MSV000087889) to determine which genera or strains produced specific amino-acid-conjugated bile acids, requiring joint agreement of m/z, retention time, drift time, and isotope dot-product.

## When NOT to use

- When the input is human stool or IBD fecal sample data rather than bacterial-culture isolates — the workflow is specific to strain-level culture screening
- When no synthesized conjugated-bile-acid standard spectra are available as a reference library, since the skill depends on searching culture spectra against those standards

## Inputs

- MSV000084475: bacterial-culture positive-mode LC-MS/MS raw data (Bruker Q-ToF)
- MSV000087889: bacterial-culture LC-IMS-MS raw data (Agilent 6560 IM-Q-ToF, negative mode)
- Synthesized conjugated-bile-acid standard MS/MS spectra (reference library)

## Outputs

- strain_conjugate_match_table.csv: strain-by-conjugate matrix with detection/match status and abundance (one row per strain-conjugate pair)

## How to apply

Download the bacterial-culture positive-mode LC-MS/MS raw data (MSV000084475, Bruker Q-ToF) and the bacterial-culture LC-IMS-MS raw data (MSV000087889, Agilent 6560 IM-Q-ToF) from MassIVE together with the synthesized conjugated-bile-acid standard spectra used as the reference library. Process MSV000084475 in MZmine2 for feature detection and alignment, then run GNPS feature-based molecular networking so each strain's chromatographic features carry its culture MS/MS spectra. Search the culture MS/MS spectra against the bile-acid standard library in GNPS to assign amino-acid conjugations and extract per-strain feature abundances; in parallel, build a targeted small-molecule method from the standards in Skyline and re-extract MSV000087889 in negative mode, requiring joint agreement of m/z, retention time, drift time and isotope dot-product (idotp) to confirm each conjugate. Finally, merge the positive-mode LC-MS/MS and ion-mobility hits into a single strain-by-conjugate matrix holding detection/match status and abundance, and write it to strain_conjugate_match_table.csv with one row per strain-conjugate pair.

## Related tools

- **MZmine2** (Feature detection and alignment on the positive-mode LC-MS/MS bacterial-culture data before GNPS molecular networking)
- **GNPS** (Feature-based molecular networking and searching culture MS/MS spectra against the bile-acid standard library to assign conjugations and extract per-strain abundances)
- **Skyline** (Targeted small-molecule method built from the synthesized standards to re-extract the LC-IMS-MS data)
- **Agilent 6560 IM-Q-ToF MS** (Instrument generating negative-mode ion-mobility MS data used for confirmation of conjugates via drift time and isotope dot-product)
- **Bruker Q-ToF** (Instrument generating positive-mode LC-MS/MS data for the bacterial-culture screen)

## Examples

```
make run
```

## Evaluation signals

- Each confirmed conjugate in the LC-IMS-MS re-extraction must show joint agreement of m/z, retention time, drift time, and isotope dot-product (idotp)
- The merged table strain_conjugate_match_table.csv must have exactly one row per strain-conjugate pair, with detection/match status and abundance populated
- Bacterial genera reported as top producers (Bifidobacterium, Enterococcus, Clostridium, Cellulosilyticum, Christensenella/Catabacter, Fusobacterium) should dominate the matrix when ranked by number of conjugated bile acids detected
- GNPS molecular networking must attach each culture MS/MS spectrum to the corresponding strain's chromatographic feature, allowing per-strain feature abundance extraction

## Limitations

- Isomer ambiguity: without isolation and NMR or X-ray structural analyses, different isomers cannot be excluded — the MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, though it was found to prevent completely inaccurate identifications
- The 1.2 billion spectra in MASST searches do not encompass all molecules, limiting re-analysis of public data in the discovery phase
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition
- The Catabacter genus was renamed Christensenella, so older strain labels may need mapping

## Evidence

- [results] Gram-positive bacteria from the Bifidobacterium, Enterococcus, Clostridium, Cellulosilyticum and Catabacter (renamed Christensenella) genera produced the most conjugated bile acids in the isolate screen.: [evidence span withheld — anchor status: not_found]
- [abstract] Culture of bacteria belonging to the Bifidobacterium, Clostridium and Enterococcus genera produced these bile amidates.: "Culture of bacteria belonging to the Bifidobacterium , Clostridium and Enterococcus genera produced these bile amidates."
- [other] The workflow step requires joint agreement of m/z, retention time, drift time, and isotope dot-product (idotp) to confirm each conjugate — this is the exact confirmation criterion used for the LC-IMS-MS re-extraction in Skyline.: [evidence span withheld — anchor status: not_found]
- [discussion] The matching protocol does not guarantee against false discovery of closely related isomers or compounds, but was found to prevent completely inaccurate identifications.: "Although this protocol does not guarantee against false discovery of closely related isomers or compounds, we found that it does prevent completely inaccurate identifications."
- [supplementary] Conjugated bile acids were detected in bacterial cultures on a Bruker Q-ToF in positive mode and on an Agilent 6560 IM-Q-ToF MS in negative mode.: "conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode"
- [discussion] The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds.: [evidence span withheld — anchor status: not_found]
