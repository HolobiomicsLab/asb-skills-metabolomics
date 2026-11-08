---
name: lipid-ion-annotation-in-mass-spectrometry-imaging
description: Use when you have a list of up- or down-regulated ion m/z values from
  an MSI experiment (e.g. rMSIKeyIon 'ions' output) and need putative metabolite identifications
  against HMDB with common positive-mode adducts at a defined ppm tolerance.
when_to_use_negative:
- The input is not a filtered, statistically significant ion m/z list (e.g. raw peak
  matrices without rMSIKeyIon TestIonSelect filtering) — run ion filtering first
- Analytes are not plausibly metabolites/lipids in positive-ion mode with H/Na/K/NH4
  adducts (e.g. negative-mode fatty acid deprotonation studies)
- The tissue of interest differs from the biological-origin filter applied (e.g. brain
  tissue would need a brain-plausibility filter rather than the liver filter)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: rMSIKeyIon
  role: Produces the statistically significant (up/down-regulated) ion list that is
    subsequently annotated against HMDB 4.0
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix and visualizes clusters/ions underlying the m/z list
    being annotated
  repo: https://github.com/prafols/rMSIproc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:d46a19bec80658bcd2e313ba41606883da926ad404e806208bf679b585fa3752
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lipid-ion-annotation-in-mass-spectrometry-imaging

## Summary

Annotates up-regulated lipid ion m/z lists from mass spectrometry imaging (e.g. cluster 2 liver ions from rMSIKeyIon TestIonSelect results) by matching them against the HMDB 4.0 database within a 20 ppm tolerance, accounting for H, Na, K, and NH4 adducts, and filtering by biological origin of the tissue.

## When to use

Use when you have a list of up- or down-regulated ion m/z values from an MSI experiment (e.g. rMSIKeyIon 'ions' output) and need putative metabolite identifications against HMDB with common positive-mode adducts at a defined ppm tolerance.

## When NOT to use

- The input is not a filtered, statistically significant ion m/z list (e.g. raw peak matrices without rMSIKeyIon TestIonSelect filtering) — run ion filtering first
- Analytes are not plausibly metabolites/lipids in positive-ion mode with H/Na/K/NH4 adducts (e.g. negative-mode fatty acid deprotonation studies)
- The tissue of interest differs from the biological-origin filter applied (e.g. brain tissue would need a brain-plausibility filter rather than the liver filter)

## Inputs

- Up-regulated ion m/z list from the 'ions' component of rMSIKeyIon TestIonSelect results (e.g. liver C2 cluster ions)
- rMSIproc peak matrix (e.g. Brain_peak_matrix.zip or liver equivalent) for context on ion intensities and clusters
- HMDB 4.0 metabolite database reference masses

## Outputs

- Annotation table (e.g. liver_c2_annotations.csv) with columns: measured m/z, adduct type, theoretical mass, mass error in ppm, HMDB identifier, metabolite name
- Set of putative metabolite identifications passing the 20 ppm tolerance and liver-origin biological filter

## How to apply

Load the up-regulated ion m/z list from the 'ions' component of the rMSIKeyIon TestIonSelect results (e.g. liver cluster 2 ions from the rMSIproc peak matrix). For each measured m/z, compute theoretical adduct masses for protonated ([M+H]+), sodium ([M+Na]+), potassium ([M+K]+), and ammonium ([M+NH4]+) forms of candidate HMDB 4.0 metabolites and accept matches within a 20 ppm mass tolerance. Apply a biological-origin filter, keeping only metabolites reported in the tissue of interest (e.g. liver) or otherwise biologically plausible for it, since HMDB alone will return many spurious database hits. Compile surviving matches into a table with measured m/z, adduct type, theoretical mass, mass error in ppm, HMDB identifier, and metabolite name, saved as a CSV such as liver_c2_annotations.csv. The adducts H, Na, K, and NH4 are chosen because they are the dominant positive-mode cation adducts observed for lipids in LDI-MS imaging of the mouse liver thirdhand-smoke study, where triacylglycerols and phosphatidylcholines predominated.

## Related tools

- **rMSIKeyIon** (Produces the statistically significant (up/down-regulated) ion list that is subsequently annotated against HMDB 4.0) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix and visualizes clusters/ions underlying the m/z list being annotated) — https://github.com/prafols/rMSIproc

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Every reported annotation must fall within the 20 ppm tolerance of its theoretical adduct mass
- Every surviving annotation must pass the liver-origin biological-origin filter (metabolite reported in liver or biologically plausible there)
- Each measured m/z maps to exactly one declared adduct type ([M+H]+, [M+Na]+, [M+K]+, or [M+NH4]+) with a consistent ppm error
- Output table liver_c2_annotations.csv contains the full schema: measured m/z, adduct type, theoretical mass, mass error ppm, HMDB ID, metabolite name
- Annotated identifications should be dominated by lipid classes (triacylglycerols and phosphatidylcholines), consistent with the study's reported findings

## Limitations

- High percentage of pixels with undetected MS ions (null peaks) reduces statistical reliability of the ion list that feeds annotation; rMSIKeyIon's zeroThreshold parameter addresses this upstream
- No discussion/limitations section text is available in the extracted article, so reproducibility gaps and failure modes of the annotation step itself cannot be fully characterized
- Match quality depends on the biological-origin filter; HMDB hits outside the tissue plausibility constraint are discarded, so genuinely novel liver metabolites may be missed

## Evidence

- [full_text] 20 ppm tolerance with H/Na/K/NH4 adducts: [evidence span withheld — anchor status: not_found]
- [full_text] Biological origin filter: [evidence span withheld — anchor status: not_found]
- [full_text] Validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] Ion list source: "The first one, called "ions" contains the up/down-regulated ions found at each comparison."
- [abstract] Lipid classes found: "and determined some lipid compounds (mainly triacylglycerols and phosphatidylcholines) in the liver of mice exposed to thirdhand smoke."
- [methods] TestIonSelect invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,"
