---
name: yeast-lipidome-background-modeling
description: Use when designing or validating a targeted PRM/MRM lipidomics assay
  and you need to quantify how many precursor-fragment pairs (0, 1, 2, 3 fragments)
  are required for unambiguous lipid species identification within a defined background
  lipidome.
when_to_use_negative:
- The sample is a complex plasma or platelet lipidome where the article's yeast-specific
  finding about fragment/polarity requirements has not been established.
- The goal is downstream quantitative processing (peak area integration, FDR scoring)
  — LipidCreator intentionally does not provide a dedicated scoring algorithm to determine
  false discovery rates or address coeluting/isobaric lipids.
- The organism is a bacterium such as E. coli or archaea, where the article notes
  standards (e.g. acyl-PG, acyl-PE in E. coli) are lacking.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipidCreator
  role: Computes MS/MS fragment masses for lipid classes and generates target/decoy
    transition lists for the background enumeration step.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts LipidCreator as a plugin to build PRM/MRM assay transition lists from
    the lipid background model.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:00b7026dec9a5f5abb606f1b78d3e6886c507d633a96a701b1856efcde9e4a21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Yeast lipidome background modeling

## Summary

Models the yeast lipidome as a background matrix of candidate lipid species and fragment masses to compute how many fragments (and which mass tolerance) are needed for unambiguous target-decoy identification, validating against the reported ~98% accuracy at three fragments in both polarity modes.

## When to use

Use when designing or validating a targeted PRM/MRM lipidomics assay and you need to quantify how many precursor-fragment pairs (0, 1, 2, 3 fragments) are required for unambiguous lipid species identification within a defined background lipidome.

## When NOT to use

- The sample is a complex plasma or platelet lipidome where the article's yeast-specific finding about fragment/polarity requirements has not been established.
- The goal is downstream quantitative processing (peak area integration, FDR scoring) — LipidCreator intentionally does not provide a dedicated scoring algorithm to determine false discovery rates or address coeluting/isobaric lipids.
- The organism is a bacterium such as E. coli or archaea, where the article notes standards (e.g. acyl-PG, acyl-PE in E. coli) are lacking.

## Inputs

- Yeast lipidome species list (background matrix of lipid species identifiers)
- In silico fragment mass enumeration per lipid species (from LipidCreator transition lists)
- Published target-decoy LC/ESI PRM validation results (true-target vs false-decoy identification counts)

## Outputs

- validation_results.csv with fragment-count-dependent ambiguity probabilities at ±0.5 Da and ±2.5 ppm tolerances
- Accuracy-ratio comparison table (computed probabilities vs reported target-decoy accuracy, e.g. 98% at three fragments)

## How to apply

Load the yeast lipidome species list as the background matrix and enumerate every candidate lipid species and its fragment masses, e.g. computed with LipidCreator's MS/MS fragment mass framework (60+ lipid classes). For each fragment count (0, 1, 2, 3), compute the probability that a random precursor-fragment pair matches unambiguously within that background at ±0.5 Da tolerance, then repeat at ±2.5 ppm to quantify the benefit of high-resolution (e.g. Q-Exactive HF PRM) filtering. Independently, load the published target-decoy LC/ESI PRM validation counts (true-target vs false-decoy identifications) and derive the accuracy ratio — the article reports all target lipids identified with only one decoy misidentified at three fragments across both polarities, i.e. 98% accuracy. Compare computed probabilities against the reported accuracy ratio; if the three-fragment result does not reproduce ~98% within stated tolerance, flag the discrepancy. The rationale is that the article's discussion establishes that multiple fragments and both polarity modes are required for unequivocal identification even in simple model systems such as yeast.

## Related tools

- **LipidCreator** (Computes MS/MS fragment masses for lipid classes and generates target/decoy transition lists for the background enumeration step.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts LipidCreator as a plugin to build PRM/MRM assay transition lists from the lipid background model.) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe  # launch stand-alone; build yeast lipidome transition lists, export for Skyline PRM assay, then compute ambiguity probabilities per fragment count (0-3) at ±0.5 Da and ±2.5 ppm
```

## Evaluation signals

- Computed accuracy ratio at three fragments across both polarity modes reproduces the reported ~98% figure within the stated tolerance; otherwise flag in validation_results.csv.
- Probability curve is monotonically non-increasing with fragment count (0 → 1 → 2 → 3 fragments) for both tolerance windows.
- ±2.5 ppm probabilities must show measurable separation from ±0.5 Da results, quantifying high-resolution filtering benefit.
- Decoy misidentification count in the reproduction matches the published result (only one decoy misidentified at three fragments).
- All yeast target lipids reported identified at three fragments with both polarities.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order of lipids, and isotope labeled standards must additionally be considered to avoid mis-annotations.
- No large-scale methods are available to cope with stereoisomers due to lack of isotope-coded standards.
- The majority of eukaryotic lipid classes are covered with some exceptions (Supplementary Data 1); the current implementation focuses on eukaryotic model systems.

## Evidence

- [discussion] multiple fragments and both polarity modes are required for unequivocal identification of lipid species in simple model systems such as yeast: "multiple fragments and both polarity modes are required for unequivocal identification of lipid species in simple model systems such as yeast"
- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [discussion] LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
- [discussion] additional information such as retention time, the elution order of lipids, as well as isotope labeled standards, if available, always need to be considered in order to avoid mis-annotations: "additional information such as retention time, the elution order of lipids, as well as isotope labeled standards, if available, always need to be considered in order to avoid mis-annotations."
- [readme] It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline.: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
