---
name: cosine-correlation-spectral-matching
description: Use when a metabolomics feature table includes raw mass spectra (m/z/intensity
  pairs) and you need to connect structurally unknown analytes to identified metabolites
  in a network where no KEGG or PubChem annotation exists.
when_to_use_negative:
- The input feature table lacks mass spectra (m/z and intensity pairs) — spectral
  similarity cannot be computed and structural similarity (PubChem Tanimoto >0.7)
  or KEGG RPAIR biochemical edges should be used instead.
- All metabolites already have biochemical (KEGG) or structural (PubChem) annotations
  — biochemical transformation and structural similarity networks alone are sufficient
  and spectral edges will be redundant under the unique-edges hierarchy.
- The data are non-metabolomic (genomic, proteomic, clinical) without spectra — use
  empirical dependency networks (Pearson, biweight, or Spearman correlations) instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: MetaMapR
  role: Shiny/R desktop tool that computes pairwise cosine correlations between mass
    spectra, applies the >0.7 cosine threshold, and exports the spectral similarity
    network as .csv for Cytoscape import.
  repo: http://dgrapov.github.io/MetaMapR/
- name: R
  role: Underlying language in which MetaMapR's spectral similarity calculation and
    Shiny interface are implemented.
  repo: http://cran.us.r-project.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:780229698417db2487ab558bfd7d305289097b94fab8796ce623b90b04c1d37b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaMapR
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cosine-correlation-spectral-matching

## Summary

Builds mass spectral similarity networks by encoding each molecular feature's mass spectrum (m/z and intensity pairs) as a vector and linking pairs whose cosine correlation exceeds a tunable threshold (e.g. >0.7), which extends biochemical/structural network analysis to structurally unannotated metabolites in MetaMapR.

## When to use

Use when a metabolomics feature table includes raw mass spectra (m/z/intensity pairs) and you need to connect structurally unknown analytes to identified metabolites in a network where no KEGG or PubChem annotation exists.

## When NOT to use

- The input feature table lacks mass spectra (m/z and intensity pairs) — spectral similarity cannot be computed and structural similarity (PubChem Tanimoto >0.7) or KEGG RPAIR biochemical edges should be used instead.
- All metabolites already have biochemical (KEGG) or structural (PubChem) annotations — biochemical transformation and structural similarity networks alone are sufficient and spectral edges will be redundant under the unique-edges hierarchy.
- The data are non-metabolomic (genomic, proteomic, clinical) without spectra — use empirical dependency networks (Pearson, biweight, or Spearman correlations) instead.

## Inputs

- Metabolomics feature table with mass spectra encoded as m/z and intensity pairs (e.g. GC-TOF-MS spectra for >470 plasma metabolites)
- Optionally: analyte-to-identifier mapping (row order determines default numeric node ID in MetaMapR)
- Cosine correlation threshold setting (e.g. >0.7)

## Outputs

- Edge list of mass spectral similarity edges (pairs of molecular features with cosine correlation > threshold) exported as a .csv file
- Network of structurally unknown metabolites linked to identified species via spectral similarity

## How to apply

Load the mass spectral data from the MetaMapR example dataset, where each molecular feature's spectrum is encoded as mass-to-charge ratio (m/z) and intensity pairs. Compute pairwise cosine correlations between all molecular features' mass spectra, yielding similarity scores bounded between 0 and 1. Filter the spectral similarity edges using the cosine correlation threshold >0.7 as used for Figures 1B and 1C — in MetaMapR this is set under the network tab (more options > mass spectral similarity threshold), and it is tunable rather than fixed. Apply the control of total edges per object and object-specific edge acceptance (e.g. limiting connections to only show annotated-to-unknown relationships) so the network stays readable. In MetaMapR the resulting edge type is ranked below biochemical and structural edges when the 'unique edges' checkbox is selected (hierarchy: Biochemical > Structural > Mass Spectral > Correlation), otherwise all edges are returned. Finally export the retained edge list as a comma separated value (.csv) file.

## Related tools

- **MetaMapR** (Shiny/R desktop tool that computes pairwise cosine correlations between mass spectra, applies the >0.7 cosine threshold, and exports the spectral similarity network as .csv for Cytoscape import.) — http://dgrapov.github.io/MetaMapR/
- **R** (Underlying language in which MetaMapR's spectral similarity calculation and Shiny interface are implemented.) — http://cran.us.r-project.org/

## Examples

```
install_github("dgrapov/MetaMapR"); # in Shiny UI: network tab > more options > mass spectral similarity threshold >0.7 > calculate network > export .csv
```

## Evaluation signals

- Output edge list exists as a .csv file.
- All retained cosine correlation scores fall within 0 to 1.
- Every retained edge exceeds the 0.7 cosine correlation threshold.
- When 'unique edges' is enabled, Mass Spectral edges only appear where no higher-priority Biochemical or Structural edge exists for the same node pair.
- Numeric node IDs in the edge list map 1:1 to the uploaded data row order (or to a custom edge index if set under network > more options).

## Limitations

- The threshold was set to >0.7 for Figures 1B & 1C, but is tunable — different thresholds will change network density and no default is specified.
- Spectral similarity extends analysis to unknowns but does not resolve molecular structure or identify unknowns.
- MetaMapR is available as web-based or desktop software written in R and requires the R package Shiny and the shiny-server for hosted deployment; the CTS API must be reachable for identifier translation steps that precede network calculation.
- KEGG does not fully describe directionality of biochemical transformations, which is why orthogonal spectral/structural edges are needed in the first place.

## Evidence

- [methods] Mass spectral similarity edges are defined as cosine correlations between mass spectra represented as vectors: "Spectral Similarity Networks are calculated based on pairwise similarities between mass spectra"
- [results] cosine correlation) which was set to >0.7 for Figures1B & 1C: "cosine correlation) which was set to >0.7 for Figures1B & 1C"
- [results] Mass spectral information can be used to extend the analysis of biochemical and structural similarity relationships to molecules without structural annotation (unknowns) (Figure 1B): [evidence span withheld — anchor status: not_found]
- [readme] The default setting for returning all edge types or only unique edges can be found under the network tab using the unique edges checkbox. If the option is selected then edges are returned based on a hierarchy Biochemical > Structural > Mass Spectral > Correlation, otherwise all edges are returned.: [evidence span withheld — anchor status: external_doc]
- [readme] By default the numeric node IDs are assigned based on the order (row) of the uploaded data. For example 1 corresponds to the analyte in the first row.: "By default the numeric node IDs are assigned based on the order (row) of the uploaded data. For example 1 corresponds to the analyte in the first row."
