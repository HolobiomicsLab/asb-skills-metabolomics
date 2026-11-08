---
name: ms2-fragment-similarity-scoring
description: Use when bridging in vivo metabolite features to in vitro TCM prototype
  components from LC-MS feature lists, and you need similarity-of-MS2-fragment evidence
  combined with m/z and mass-defect scoring to assign metabolites to prototypes.
when_to_use_negative:
- When only single-metabolite-to-prototype-database matching on secondary mass spectra
  is needed — the article notes MTSF already does this and MDRB adds m/z and mass-defect
  dimensions
- When the input is raw MS data rather than a processed feature list with m/z, mass
  defects, and MS2 fragment spectra from a non-targeted identification step (PATBS)
- When metabolite structures must be predicted de novo without any prototype database
  — MDRB bridges metabolites to known prototypes and does not identify unbridged components
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MDRB
  role: R-based data post-processing pipeline that sequentially analyzes m/z, MS1/MS2
    mass defect, and MS2 fragment similarity to score and bridge metabolite-prototype
    pairs
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Produces the non-targeted in vivo metabolite feature list and in vitro prototype
    list that feed the scoring/bridging step
- name: Metworks 1.3.0.200
  role: Supplies the known predictable metabolic reaction definitions used to enumerate
    one- and two-step predictable metabolites before MS2 similarity scoring
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:0a60efe3c2a7c25c12ab3af407bb6c6b4e7457b8f5dcff620c543b33b8b1e75a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# ms2-fragment-similarity-scoring

## Summary

Scores metabolite-prototype candidate pairs within the MDRB strategy using MS2 fragment similarity as one of three dimensions of mass spectral evidence, combined with m/z (±5 ppm, 70 points) and MS1/MS2 mass defect bridging to assign metabolites to TCM prototypes.

## When to use

Use when bridging in vivo metabolite features to in vitro TCM prototype components from LC-MS feature lists, and you need similarity-of-MS2-fragment evidence combined with m/z and mass-defect scoring to assign metabolites to prototypes.

## When NOT to use

- When only single-metabolite-to-prototype-database matching on secondary mass spectra is needed — the article notes MTSF already does this and MDRB adds m/z and mass-defect dimensions
- When the input is raw MS data rather than a processed feature list with m/z, mass defects, and MS2 fragment spectra from a non-targeted identification step (PATBS)
- When metabolite structures must be predicted de novo without any prototype database — MDRB bridges metabolites to known prototypes and does not identify unbridged components

## Inputs

- Metabolite feature list with m/z, MS1 and MS2 mass defects, and MS2 fragment spectra (from PATBS non-targeted identification)
- In vitro prototype (CKXR component) list from PATBS
- Known predictable metabolic reaction definitions from Metworks 1.3.0.200 plus TCM deglycosylation reactions

## Outputs

- Scored metabolite-prototype assignment table (e.g. predictable_reaction_catalog_scores.csv) with m/z (70 pts) and MS2 similarity (20/30/0 pts) components
- Component relationship network bridging in vivo metabolites to in vitro prototypes across plasma, urine, feces, and lung matrices
- Characterized metabolite counts per matrix (reported: 66 plasma, 222 urine, 30 feces, 40 lung; 227 total, 65%)

## How to apply

Load the CKXR metabolite feature list (m/z, MS1/MS2 mass defects, MS2 fragment spectra) and the in vitro prototype list from the PATBS non-targeted identification step, then run the MDRB R pipeline (https://github.com/933ZhangDD/MDRB), which sequentially analyzes m/z, mass defect of MS1 and MS2, and MS2 fragment similarity. For each metabolite-prototype pair, award 70 points when the m/z matches within ±5 ppm. Score MS2 fragment similarity per match degree: 20 points for match degree 1, 30 points for match degree ≥2, and 0 points when no match is found; combine with the m/z component and the mass-defect-based structural-similarity/minimal-mass-defect-variation score into an overall bridging score. For predictable metabolites, restrict candidates to one- or two-step transformation products from the Metworks 1.3.0.200 reaction catalog plus TCM deglycosylation reactions before scoring. Output the scored assignments (e.g. predictable_reaction_catalog_scores.csv) and the component relationship network linking plasma, urine, feces, and lung metabolites to prototypes.

## Related tools

- **MDRB** (R-based data post-processing pipeline that sequentially analyzes m/z, MS1/MS2 mass defect, and MS2 fragment similarity to score and bridge metabolite-prototype pairs) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Produces the non-targeted in vivo metabolite feature list and in vitro prototype list that feed the scoring/bridging step)
- **Metworks 1.3.0.200** (Supplies the known predictable metabolic reaction definitions used to enumerate one- and two-step predictable metabolites before MS2 similarity scoring)

## Examples

```
git clone https://github.com/933ZhangDD/MDRB && Rscript MDRB.R  # run in R on PATBS-derived metabolite and prototype feature lists to score pairs and write predictable_reaction_catalog_scores.csv
```

## Evaluation signals

- m/z assignment score equals 70 only when the metabolite-prototype m/z matches within ±5 ppm; verify per-pair scores in the output table
- MS2 similarity component is exactly 20, 30, or 0 points corresponding to match degree 1, ≥2, or no match — check no other values appear
- Predictable reaction catalog contains both Metworks 1.3.0.200-derived reactions and the added TCM deglycosylation reactions
- Per-matrix characterized counts reproduce 66/222/30/40 and the total matches 227 metabolites (65%) versus the manual 110 (31%) baseline
- Component relationship network links in vivo metabolites to in vitro prototypes in all four matrices (plasma, urine, feces, lung)

## Limitations

- MDRB only utilizes the similarity of secondary mass spectrometry maps as one dimension; pairs lacking MS2 fragment matches score 0 on that component and rely on m/z and mass-defect evidence
- No specific mass spectrometry instrument model or data acquisition parameters are named, complicating direct reproduction of acquisition-dependent MS2 similarity
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy beyond the 227 vs 110 characterization comparison
- Identification of in vivo components remains difficult despite MS data post-processing techniques

## Evidence

- [abstract] The current method integrates three dimensions of mass: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [abstract] scoring based on structural similarity: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes"
- [results] MS2 similarity degree scoring: [evidence span withheld — anchor status: not_found]
- [abstract] MDRB repository availability: "the developed MDRB platform is publicly available for ( https://github.com/933ZhangDD/MDRB )"
- [intro] MS2 fragment mass defect bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites"
