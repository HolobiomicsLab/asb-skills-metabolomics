---
name: molecular-network-analog-prioritization
description: Use when a molecular network (e.g., GNPS) contains an unannotated node
  adjacent to library hits of a related metabolite, and the structural site of modification
  between the analogs must be localized from MS/MS spectra.
when_to_use_negative:
- The unknown compound can be directly reidentified by standard MS/MS library search
  against GNPS libraries (no analog step is needed)
- No library-hit analog is adjacent in the molecular network — ModiFinder requires
  a structurally related known molecule as reference
- Precursor masses differ by more than 50% or exceed 2000 Da (benchmark eligibility
  checks i–ii), making analog alignment unreliable
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: ModiFinder
  role: Aligns MS/MS peaks between known and unknown analogs and scores per-atom likelihood
    of the modification site; available as CLI and web interface
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: molecular networking
  role: Identifies the molecular family of adjacent unknown and library-hit nodes
    that supplies the analog pair for localization
- name: Analog library search
  role: Extends library identification to unknown compounds by matching them to related
    known structures within the network
- name: SIRIUS
  role: Provides predicted formulas used to filter possible substructures for each
    MS/MS peak (Substructure Refinement By Formula)
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:a951fdad03f0e1135f607a2bdf6637a7767c813147780f5fd9fda58e7f78afb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - Analog library search
  - ModiFinder
  - SIRIUS
  - molecular networking
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# molecular-network-analog-prioritization

## Summary

Localizes the modification site of an unknown analog within a molecular network family by aligning its MS/MS spectrum against a structurally related library-hit analog using ModiFinder, which scores the likelihood of each atom in the known analog being the modification site.

## When to use

Use when a molecular network (e.g., GNPS) contains an unannotated node adjacent to library hits of a related metabolite, and the structural site of modification between the analogs must be localized from MS/MS spectra.

## When NOT to use

- The unknown compound can be directly reidentified by standard MS/MS library search against GNPS libraries (no analog step is needed)
- No library-hit analog is adjacent in the molecular network — ModiFinder requires a structurally related known molecule as reference
- Precursor masses differ by more than 50% or exceed 2000 Da (benchmark eligibility checks i–ii), making analog alignment unreliable

## Inputs

- MS/MS spectra (.mgf) of the unknown analog node from a molecular network family
- MS/MS spectra of structurally related library-hit analogs (e.g., MSV/GNPS library entries)
- Candidate chemical structures (SMILES/MOL) of the known analogs
- Molecular network GNPS/MassIVE deposit containing the target molecular family

## Outputs

- Per-atom likelihood scores for the modification site on the known analog (e.g., modifinder_localization.csv)
- Top-ranked localization region for each analog comparison
- Heatmap visualization of predicted modification sites via draw_prediction

## How to apply

Retrieve the molecular family containing the unknown node and its neighboring library hits, then preprocess each MS/MS spectrum by removing peaks below 1% of the base peak and normalizing intensities to a Euclidean norm of 1. Run ModiFinder treating the unknown as the query and a known analog as the reference: it performs GNPS cosine peak alignment (40 ppm tolerance), assigns possible substructures to peaks via MAGMa-style combinatorial fragmentation (breadth-first, maximum depth 2), and increases atom likelihood scores for shifted peaks while decreasing them for unshifted peaks. Repeat the run against each candidate analog in the family (e.g., comparing against different headgroup or lipid-tail analogs) to test competing structural hypotheses. Visualize per-atom likelihoods with ModiFinder's draw_prediction heatmap over the analog structures and export the scores for the top-ranked localization region.

## Related tools

- **ModiFinder** (Aligns MS/MS peaks between known and unknown analogs and scores per-atom likelihood of the modification site; available as CLI and web interface) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **molecular networking** (Identifies the molecular family of adjacent unknown and library-hit nodes that supplies the analog pair for localization)
- **Analog library search** (Extends library identification to unknown compounds by matching them to related known structures within the network)
- **SIRIUS** (Provides predicted formulas used to filter possible substructures for each MS/MS peak (Substructure Refinement By Formula)) — https://github.com/boecker-lab/sirius

## Evaluation signals

- Localized modification is consistent with the structural hypothesis: e.g., a shortened lipid tail relative to Glu-C16:1 and a modified amine headgroup relative to Thr-C12:1, jointly matching the manually verified true structure
- Post-processed likelihood scores show a focused high-scoring atom region (values below 0.5 set to 0, remaining values raised to the power of 4 to highlight top atoms)
- Predicted substructure formulas fall within the 40 ppm m/z tolerance of the assigned empirical peaks
- Preprocessed spectra contain no peaks below 1% of the base peak and have Euclidean norm 1
- Repeat runs against different analogs (Glu-C16:1 vs Thr-C12:1) converge on compatible modification regions

## Limitations

- Performance depends on the availability and structural relatedness of a library-hit analog in the same molecular family; without a suitable analog the localization cannot be performed
- ModiFinder's project README states the project is under active development
- Software is licensed for academic/nonprofit research use only; commercial (re)distribution is prohibited, and SIRIUS web services are restricted to academic research and education

## Evidence

- [readme] Tool purpose from README: [evidence span withheld — anchor status: not_found]
