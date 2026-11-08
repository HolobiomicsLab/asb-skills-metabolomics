---
name: tissue-specific-metabolite-filtering
description: Use when a list of statistically significant up/down-regulated ion m/z
  values from an MSI region comparison needs putative metabolite IDs restricted to
  those plausible for the imaged tissue.
when_to_use_negative:
- When the ion list has not been filtered by rMSIKeyIon first — annotation of an unfiltered
  peak table will not target differentially regulated ions between ROIs
- When metabolites from any biological origin are acceptable and no tissue-specificity
  constraint is desired
- When mass accuracy exceeds the 20 ppm tolerance achievable here (e.g., low-resolution
  data) — matches would be unreliable
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: rMSIKeyIon
  role: Produces the up/down-regulated ion list per ROI comparison (TestIonSelect)
    whose m/z values are then matched against HMDB
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix and visualizes clustering and ions feeding the ion list
  repo: https://github.com/prafols/rMSIproc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:9a5f84538e7d309f117c3802d78f4dc5cf8502d62950c159d1e1aec1ad4923c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tissue-specific-metabolite-filtering

## Summary

Annotates up- or down-regulated ion lists from rMSIKeyIon ROI comparisons by matching m/z values against HMDB 4.0 with common positive-mode adducts at a 20 ppm tolerance, then retaining only metabolites biologically plausible for the tissue under study (e.g., liver).

## When to use

Use when a list of statistically significant up/down-regulated ion m/z values from an MSI region comparison needs putative metabolite IDs restricted to those plausible for the imaged tissue.

## When NOT to use

- When the ion list has not been filtered by rMSIKeyIon first — annotation of an unfiltered peak table will not target differentially regulated ions between ROIs
- When metabolites from any biological origin are acceptable and no tissue-specificity constraint is desired
- When mass accuracy exceeds the 20 ppm tolerance achievable here (e.g., low-resolution data) — matches would be unreliable

## Inputs

- Up/down-regulated ion m/z list from the 'ions' component of rMSIKeyIon TestIonSelect results (R list)
- rMSIproc peak matrix (e.g., Brain_peak_matrix.zip or tissue-specific peak_matrix.Rdata) used upstream to derive the ion list
- HMDB 4.0 metabolite database

## Outputs

- Annotation table (CSV, e.g. liver_c2_annotations.csv) with columns: measured m/z, adduct type, theoretical mass, mass error in ppm, HMDB identifier, metabolite name
- Set of tissue-plausible putative metabolite identifications per adduct

## How to apply

Take the up-regulated ion m/z list from the 'ions' component of the rMSIKeyIon TestIonSelect results (e.g., cluster 2 vs. other clusters in mouse liver). For each measured m/z, compute candidate neutral masses for the [M+H]+, [M+Na]+, [M+K]+, and [M+NH4]+ adducts and search them against the HMDB 4.0 database, accepting matches within a 20 ppm mass tolerance. Apply a biological-origin filter that keeps only metabolites reported in the target tissue (liver) or otherwise biologically plausible for it, which is the key decision point separating this from a generic database search. Compile surviving annotations into a table (measured m/z, adduct type, theoretical mass, ppm error, HMDB ID, metabolite name) and save as e.g. liver_c2_annotations.csv.

## Related tools

- **rMSIKeyIon** (Produces the up/down-regulated ion list per ROI comparison (TestIonSelect) whose m/z values are then matched against HMDB) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix and visualizes clustering and ions feeding the ion list) — https://github.com/prafols/rMSIproc

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Every reported annotation's |ppm error| between measured m/z and its theoretical adduct mass ([M+H]+, [M+Na]+, [M+K]+, or [M+NH4]+) must be ≤ 20 ppm
- Each metabolite in the output table passes the liver-origin biological filter (reported in liver or biologically plausible for liver tissue)
- Each annotated m/z in the output traceable to an ion in the upstream rMSIKeyIon 'ions' list
- Output CSV contains the required columns: measured m/z, adduct type, theoretical mass, mass error in ppm, HMDB identifier, metabolite name
- Reported identifications should be consistent with known tissue lipid classes (e.g., triacylglycerols and phosphatidylcholines in mouse liver)

## Limitations

- Annotations are putative only — m/z matching at 20 ppm with four adducts can yield multiple candidate metabolites per ion; tissue-plausibility filtering is heuristic
- Undetected MS signals (null peaks) in a high percentage of pixels undermine reliability of the upstream statistical tests feeding the ion list
- The rMSIKeyIon package requires the GSL library, which can complicate installation (especially on Windows)

## Evidence

- [other] matched against HMDB with liver plausibility: [evidence span withheld — anchor status: not_found]
- [abstract] lipid classes found in liver: "and determined some lipid compounds (mainly triacylglycerols and phosphatidylcholines) in the liver of mice exposed to thirdhand smoke."
- [readme] TestIonSelect call: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] ions list structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison."
- [abstract] filtering parameters: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
