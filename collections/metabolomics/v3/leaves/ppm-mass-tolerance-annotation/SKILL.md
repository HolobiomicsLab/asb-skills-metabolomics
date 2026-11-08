---
name: ppm-mass-tolerance-annotation
description: Use when you have a list of significant/up-regulated ion m/z values from
  an rMSIKeyIon TestIonSelect comparison and need putative metabolite IDs via ppm-tolerant
  database matching with adduct consideration.
when_to_use_negative:
- Input is already an annotated feature table with metabolite names — re-annotation
  at 20 ppm would add nothing
- The ion list has not been filtered by TestIonSelect; matching unfiltered peak matrices
  will flood results with spurious candidates
- The analysis targets negative-mode or unusual adduct chemistry not covered by the
  H/Na/K/NH4 positive-mode adduct set
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: rMSIKeyIon
  role: Produces the statistically filtered up/down-regulated ion m/z list (TestIonSelect)
    that serves as the input to the ppm-tolerance database matching
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix and visualizes clustering and ions; supplies the peak
    matrix underlying the ion list
  repo: https://github.com/prafols/rMSIproc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:99e81b196a37ab5f1fb3216b562f03d7308dc98c9ff2761847db42842adc6e73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ppm-mass-tolerance-annotation

## Summary

Annotate putative metabolite identities for up-regulated MSI ion m/z lists by matching against a metabolite database (e.g. HMDB 4.0) within a fixed ppm mass tolerance, enumerating candidate adducts. This converts statistically filtered ion lists from rMSIKeyIon into named metabolites.

## When to use

Use when you have a list of significant/up-regulated ion m/z values from an rMSIKeyIon TestIonSelect comparison and need putative metabolite IDs via ppm-tolerant database matching with adduct consideration.

## When NOT to use

- Input is already an annotated feature table with metabolite names — re-annotation at 20 ppm would add nothing
- The ion list has not been filtered by TestIonSelect; matching unfiltered peak matrices will flood results with spurious candidates
- The analysis targets negative-mode or unusual adduct chemistry not covered by the H/Na/K/NH4 positive-mode adduct set

## Inputs

- List of up/down-regulated ion m/z values from the 'ions' component of rMSIKeyIon TestIonSelect results (results.Rdata or equivalent)
- Reference metabolite database (HMDB 4.0) with monoisotopic neutral masses
- Tissue context for the biological-origin filter (e.g. mouse liver)

## Outputs

- Annotation table (e.g. liver_c2_annotations.csv) with columns: measured m/z, adduct type, theoretical mass, mass error in ppm, HMDB identifier, metabolite name
- Set of putative metabolite identifications (e.g. triacylglycerols and phosphatidylcholines in mouse liver)

## How to apply

Load the significant ion m/z list from the 'ions' component of the rMSIKeyIon TestIonSelect results (e.g. the up-regulated cluster 2 liver ions). For each measured m/z, compute candidate metabolite neutral masses for common positive-mode adducts — protonated [M+H]+, sodium [M+Na]+, potassium [M+K]+, and ammonium [M+NH4]+ — and search them against HMDB 4.0, accepting matches within a 20 ppm mass tolerance. Apply a biological-origin filter, retaining only metabolites reported in or plausible for the tissue under study (liver in the thirdhand-smoke mouse study, where the identified compounds were mainly triacylglycerols and phosphatidylcholines). Compile surviving matches into a table with measured m/z, adduct type, theoretical mass, mass error in ppm, database identifier, and metabolite name.

## Related tools

- **rMSIKeyIon** (Produces the statistically filtered up/down-regulated ion m/z list (TestIonSelect) that serves as the input to the ppm-tolerance database matching) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix and visualizes clustering and ions; supplies the peak matrix underlying the ion list) — https://github.com/prafols/rMSIproc

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Every reported annotation's mass error must fall within the 20 ppm tolerance of its theoretical adduct mass
- Every surviving metabolite must pass the biological-origin filter (reported in liver or biologically plausible for liver tissue)
- Each annotated m/z should map to exactly one declared adduct type ([M+H]+, [M+Na]+, [M+K]+, or [M+NH4]+) with a consistent theoretical mass
- Output table must contain the full schema: measured m/z, adduct type, theoretical mass, ppm error, HMDB ID, metabolite name
- Identified compound classes should be consistent with prior findings (mainly triacylglycerols and phosphatidylcholines in the thirdhand-smoke mouse liver study)

## Limitations

- A 20 ppm tolerance on low-resolution LDI-MS m/z values yields putative, not confirmed, identifications — multiple isobaric metabolites and adduct combinations can match
- The adduct set considered is limited to H, Na, K, and NH4; other adducts or in-source fragments will be missed
- High percentages of undetected MS ions (null peaks) in pixels complicate upstream statistics, which propagates uncertainty into the ion list being annotated

## Evidence

- [other] 20 ppm tolerance with H/Na/K/NH4 adducts matching against HMDB 4.0: [evidence span withheld — anchor status: not_found]
- [abstract] lipid compounds determined in mouse liver: "and determined some lipid compounds (mainly triacylglycerols and phosphatidylcholines) in the liver of mice exposed to thirdhand smoke."
- [readme] TestIonSelect output list structure with 'ions' component: "The first one, called "ions" contains the up/down-regulated ions found at each comparison."
- [methods] TestIonSelect call producing the filtered ion list: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
