---
name: adduct-mass-calculation-positive-mode
description: Use when a list of significant ion m/z values (e.g. up/down-regulated
  ions from an rMSIKeyIon 'ions' output list) must be assigned putative metabolite
  identities in positive-ion LDI/MSI data.
when_to_use_negative:
- The input is an MS/MS spectrum requiring fragmentation-based identification rather
  than accurate-mass database matching
- The ion list comes from a clustering that has not yet been filtered by rMSIKeyIon,
  so it contains non-significant peaks
- Analysis is in negative-ion mode or requires adducts beyond H, Na, K, and NH4, which
  are outside the defined adduct set
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rMSIKeyIon
  role: Produces the up/down-regulated ion lists (via TestIonSelect) whose m/z values
    are then annotated against HMDB 4.0 with H/Na/K/NH4 adducts at 20 ppm
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix and visualizes clustering and ion images for the MSI
    dataset feeding the annotation workflow
  repo: https://github.com/prafols/rMSIproc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:6a5255c78093b072dfcc5123dae05c38245b2d84b0f3cf4a78c136c7e29f3b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - rMSIKeyIon
  - rMSIproc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# adduct-mass-calculation-positive-mode

## Summary

Annotates up-regulated ion m/z values from rMSIKeyIon MSI analyses by matching them against HMDB 4.0 while computing candidate neutral masses for positive-mode adducts [M+H]+, [M+Na]+, [M+K]+, and [M+NH4]+ within a 20 ppm mass tolerance, then applying a biological-origin filter.

## When to use

Use when a list of significant ion m/z values (e.g. up/down-regulated ions from an rMSIKeyIon 'ions' output list) must be assigned putative metabolite identities in positive-ion LDI/MSI data.

## When NOT to use

- The input is an MS/MS spectrum requiring fragmentation-based identification rather than accurate-mass database matching
- The ion list comes from a clustering that has not yet been filtered by rMSIKeyIon, so it contains non-significant peaks
- Analysis is in negative-ion mode or requires adducts beyond H, Na, K, and NH4, which are outside the defined adduct set

## Inputs

- Numeric m/z list of up/down-regulated ions (the 'ions' list from rMSIKeyIon::TestIonSelect output)
- HMDB 4.0 metabolite database (metabolite names and monoisotopic neutral masses)

## Outputs

- Annotation table (e.g. liver_c2_annotations.csv) with columns: measured m/z, adduct type, theoretical mass, mass error (ppm), HMDB identifier, metabolite name

## How to apply

Take the m/z values from the 'ions' component of the rMSIKeyIon::TestIonSelect results (e.g. the up-regulated cluster 2 liver ion list). For each measured m/z, compute theoretical adduct masses for the protonated, sodium, potassium, and ammonium adducts over HMDB 4.0 metabolite entries and accept matches within a 20 ppm mass tolerance. Retain only candidates that pass a biological-origin filter (e.g. reported in liver or biologically plausible for the imaged tissue), consistent with the article's approach of determining lipid compounds such as triacylglycerols and phosphatidylcholines in mouse liver. Compile surviving annotations into a table with measured m/z, adduct type, theoretical mass, ppm error, HMDB identifier, and metabolite name (e.g. liver_c2_annotations.csv).

## Related tools

- **rMSIKeyIon** (Produces the up/down-regulated ion lists (via TestIonSelect) whose m/z values are then annotated against HMDB 4.0 with H/Na/K/NH4 adducts at 20 ppm) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix and visualizes clustering and ion images for the MSI dataset feeding the annotation workflow) — https://github.com/prafols/rMSIproc

## Evaluation signals

- Every reported annotation falls within the 20 ppm mass tolerance of its theoretical adduct mass
- Each annotation is assigned to one of the four allowed adducts: [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+
- Every retained metabolite passes the liver-origin biological filter (reported in liver or biologically plausible for liver tissue)
- Output table contains the required columns: measured m/z, adduct type, theoretical mass, ppm error, HMDB ID, and metabolite name

## Limitations

- Assignments are putative: accurate-mass matching within 20 ppm without MS/MS can yield multiple candidates per m/z, so a biological-origin filter is required to prune them
- Only the H, Na, K, and NH4 adducts are considered; other positive-mode adducts (e.g. [M+NH4-H2O]+) or in-source fragments are not covered
- The article notes a high number of undetected MS ions in many pixels, which affects the reliability of the statistical tests upstream of this annotation step

## Evidence

- [full_text] matched against the HMDB 4.0 database within a 20 ppm tolerance, considering the possible ion adducts H, Na, K, and NH4: [evidence span withheld — anchor status: not_found]
- [full_text] Apply the biological-origin filter, keeping only metabolites reported in liver or otherwise biologically plausible for liver tissue: [evidence span withheld — anchor status: not_found]
- [methods] results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,"
- [readme] The first one, called "ions" contains the up/down-regulated ions found at each comparison.: "The first one, called "ions" contains the up/down-regulated ions found at each comparison."
- [abstract] determined some lipid compounds (mainly triacylglycerols and phosphatidylcholines) in the liver of mice exposed to thirdhand smoke.: "determined some lipid compounds (mainly triacylglycerols and phosphatidylcholines) in the liver of mice exposed to thirdhand smoke."
