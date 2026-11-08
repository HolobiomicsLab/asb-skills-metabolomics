---
name: natural-product-structure-annotation
description: Use when molecular networking or analog library search (GNPS) surfaces
  an unknown MS/MS spectrum with a structurally related library hit, and the modification
  site on the known structure must be localized to annotate the new natural product.
when_to_use_negative:
- No structurally related known compound (library hit or analog) exists for the unknown
  — ModiFinder requires a known reference structure for spectral alignment
- The goal is de novo molecular formula or structure elucidation from fragmentation
  trees alone (use SIRIUS directly) rather than localizing a modification between
  two related structures
- The unknown and known differ by more than the benchmark-supported regime (precursor
  mass > 2000 Da or precursor mass difference > 50%)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: ModiFinder
  role: 'Core tool: aligns MS/MS peaks between the known and unknown analog, assigns
    substructures via combinatorial fragmentation (depth 2, 40 ppm), scores per-atom
    modification-site likelihood, and renders draw_prediction heatmaps; available
    as a command line tool and interactive web interface'
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: molecular networking
  role: Surfaces the molecular family containing the unknown node and its neighboring
    library-hit analogs that serve as the known reference for ModiFinder
- name: Analog library search
  role: Identifies library hits structurally related to the unknown compound, providing
    candidate known-analog spectra and structures for the alignment
- name: SIRIUS
  role: Provides predicted molecular formulas used to refine/filter the possible substructures
    assigned to each MS/MS peak; the paper passes an mgf file to the v5.6.3 runnable
    script
  repo: https://github.com/boecker-lab/sirius
- name: MAGMa
  role: Combinatorial fragmentation approach (breadth-first, maximum depth 2) that
    ModiFinder uses to assign sets of possible substructures to each peak
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:cbad73eb08ecf7a17129fe0b4c71d52325192d873178b2ef05e9222f176788ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - Analog library search
  - MAGMa
  - ModiFinder
  - SIRIUS
  - molecular networking
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# natural-product-structure-annotation

## Summary

Localize the structural modification site of an unknown natural product analog relative to a known library hit by aligning their MS/MS spectra with ModiFinder, which scores the likelihood of each atom in the known molecule as the modification site.

## When to use

Use when molecular networking or analog library search (GNPS) surfaces an unknown MS/MS spectrum with a structurally related library hit, and the modification site on the known structure must be localized to annotate the new natural product.

## When NOT to use

- No structurally related known compound (library hit or analog) exists for the unknown — ModiFinder requires a known reference structure for spectral alignment
- The goal is de novo molecular formula or structure elucidation from fragmentation trees alone (use SIRIUS directly) rather than localizing a modification between two related structures
- The unknown and known differ by more than the benchmark-supported regime (precursor mass > 2000 Da or precursor mass difference > 50%)

## Inputs

- MS/MS spectrum of the unknown compound (e.g., 328.211 m/z node from a GNPS molecular network)
- MS/MS spectrum and 2D structure of a structurally related known library hit (e.g., Glu-C16:1 or Thr-C12:1)
- Precursor m/z values for both known and unknown compounds
- Optional: predicted molecular formulas from SIRIUS or BUDDY for substructure refinement
- Molecular family membership from GNPS molecular networking / analog library search

## Outputs

- Per-atom likelihood scores for the modification site on the known analog structure
- Top-ranked modification-site localization region on the known structure
- draw_prediction heatmap visualization over the known analog structure
- CSV of per-atom scores and predicted localization (e.g., modifinder_328211_localization.csv)

## How to apply

Retrieve the molecular family containing the unknown node and its neighboring library hits (e.g., the 328.211 m/z analog with Thr-C12:1 and Glu-C16:1 neighbors in the Synechococcus elongatus network). Preprocess each MS/MS spectrum by removing peaks below 1% of the base peak and normalizing intensities to a Euclidean norm of 1. Run ModiFinder treating the unknown as the query and one library hit as the known analog: peaks are aligned via bipartite graph greedy matching (GNPS cosine alignment, 40 ppm tolerance), substructures are assigned by MAGMa-style combinatorial fragmentation at a maximum depth of 2, shifted peaks raise likelihood scores of atoms in assigned substructures while unshifted peaks lower them. Optionally refine candidate substructures using predicted formulas from SIRIUS or BUDDY. Post-process scores by zeroing values below 0.5 and raising the rest to the power of 4, then visualize with ModiFinder's draw_prediction heatmap and export per-atom likelihood scores (e.g., modifinder_328211_localization.csv). Rerun with alternative known analogs to test competing hypotheses (lipid-tail shortening vs. headgroup modification) and check that the top-ranked region is chemically consistent.

## Related tools

- **ModiFinder** (Core tool: aligns MS/MS peaks between the known and unknown analog, assigns substructures via combinatorial fragmentation (depth 2, 40 ppm), scores per-atom modification-site likelihood, and renders draw_prediction heatmaps; available as a command line tool and interactive web interface) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **molecular networking** (Surfaces the molecular family containing the unknown node and its neighboring library-hit analogs that serve as the known reference for ModiFinder)
- **Analog library search** (Identifies library hits structurally related to the unknown compound, providing candidate known-analog spectra and structures for the alignment)
- **SIRIUS** (Provides predicted molecular formulas used to refine/filter the possible substructures assigned to each MS/MS peak; the paper passes an mgf file to the v5.6.3 runnable script) — https://github.com/boecker-lab/sirius
- **MAGMa** (Combinatorial fragmentation approach (breadth-first, maximum depth 2) that ModiFinder uses to assign sets of possible substructures to each peak)

## Evaluation signals

- Per-atom likelihood scores are exported and the top-ranked localization region is consistent with the chemically expected modification (e.g., shortened lipid tail on Glu-C16:1 and modified amine headgroup relative to Thr-C12:1, jointly matching Glu-C12:1)
- Preprocessing invariant: all peaks with intensity < 1% of the base peak are removed and remaining peaks are normalized to a Euclidean norm of 1
- Substructure-to-peak assignments respect the 40 ppm m/z tolerance and fragmentation does not exceed the maximum depth of 2
- Post-processed score landscape: values below 0.5 are set to 0 and remaining values raised to the power of 4, so high-scoring atoms are clearly separated
- Independent analog comparison: rerunning with a second known analog (e.g., Thr-C12:1 vs Glu-C16:1) yields a localization that supports the same overall structure hypothesis

## Limitations

- ModiFinder scores require a structurally related known compound; the approach extends analog search and molecular networking but does not identify compounds de novo
- Benchmark pair eligibility restricts precursor mass to < 2000 Da and precursor mass difference to < 50%; helper refinement eliminates compounds with precursor mass within a 0.5 range of the unknown
- Oracle substructure filtering (removing substructures not containing the true modification site) is only available for benchmark evaluation, not for real discovery use
- The ModiFinder_base repository states the project 'is under active development' and is licensed for academic/nonprofit research use only

## Evidence

- [readme] Tool purpose: [evidence span withheld — anchor status: not_found]
