---
name: spectral-cosine-similarity-scoring
description: Use when raw LC-MS/MS metabolomics spectra (e.g. deposited mzML/MassIVE
  MSV000083738) must be grouped into molecular families by spectral similarity, or
  when correlating those families to BiG-SCAPE-derived GCFs.
when_to_use_negative:
- Input is already a BGC/GCF similarity network from BiG-SCAPE — this skill operates
  on MS/MS spectra, not gene clusters
- Only single-strain or non-deposited MS/MS data are available — the study notes MS/MS
  data for other strains is available only upon request
- The goal is GCF phylogenetic analysis (use CORASON/BiG-SCAPE alignment-based clustering
  instead of spectral scoring)
edam_operation: http://edamontology.org/operation_3216
tools:
- name: Cytoscape
  role: Visualization of the cosine-similarity molecular network as connected components
    (molecular families) for manual family identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:2b14d5cf4ee3306a7146dd390947c1192203a4db9d73ee360a263fd29b0f7436
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cytoscape
  edam_operation: http://edamontology.org/operation_3216
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-cosine-similarity-scoring

## Summary

Scores pairwise MS/MS spectral similarity with cosine metrics to link metabolite features into molecular families, then maps families to biosynthetic gene cluster families (GCFs) via metabologenomics across 363 actinomycete strains. In this study it was used to recover the 110-member detoxin/rimosamide molecular family from MassIVE MSV000083738 LC-MS/MS data.

## When to use

Use when raw LC-MS/MS metabolomics spectra (e.g. deposited mzML/MassIVE MSV000083738) must be grouped into molecular families by spectral similarity, or when correlating those families to BiG-SCAPE-derived GCFs.

## When NOT to use

- Input is already a BGC/GCF similarity network from BiG-SCAPE — this skill operates on MS/MS spectra, not gene clusters
- Only single-strain or non-deposited MS/MS data are available — the study notes MS/MS data for other strains is available only upon request
- The goal is GCF phylogenetic analysis (use CORASON/BiG-SCAPE alignment-based clustering instead of spectral scoring)

## Inputs

- LC-MS/MS raw data files from the 363-strain metabolomic study (MassIVE accession MSV000083738)
- MS/MS scans with peak intensity lists per ion
- Known detoxin and rimosamide reference spectra / MIBiG entries BGC0001840, BGC0001878, BGC0001841

## Outputs

- Cosine-similarity edge/node tables of the molecular network
- Molecular network in graphml format importable into Cytoscape
- detoxin_rimosamide_family_report.csv with the 110-member family composition (8 detoxins / 4 rimosamides / 99 novel analogues)

## How to apply

Preprocess each MS/MS scan by discarding the 25% of ions with the lowest intensity, as in the study's metabolomic pipeline. Compute pairwise cosine similarity between spectra; for ion identity the study required accurate mass within 4 ppm and fragmentation cosine similarity above 0.75 (yielding 5,824 ions in two or more strains), and molecular network edges were retained at cosine similarity >= 0.6. Import the resulting node/edge tables into Cytoscape so that molecular families appear as connected components. Identify the family of interest (here detoxins/rimosamides, referenced by MIBiG BGC0001840, BGC0001878, BGC0001841) by manually inspecting the network for ions related to known compounds. Write the family composition — 8 known detoxins, 4 known rimosamides, 99 putatively novel analogues (110 total) — to a report file and export the network as graphml.

## Related tools

- **Cytoscape** (Visualization of the cosine-similarity molecular network as connected components (molecular families) for manual family identification)

## Evaluation signals

- Exported molecular network exists in graphml format and opens as connected components in Cytoscape
- detoxin_rimosamide_family_report.csv reproduces the 110-member family with the 8/4/99 breakdown (8 known detoxins, 4 known rimosamides, 99 putatively novel analogues)
- Edge list respects the cosine similarity >= 0.6 cutoff used to build the network
- Ion-identity calls honor the 4 ppm mass and cosine > 0.75 thresholds, approximating the study's 5,824 ions detected in two or more strains
- Lowest-intensity 25% of ions removed from each MS/MS scan before scoring

## Limitations

- Molecular network family identification was performed manually, introducing curator subjectivity
- MS/MS data for non-detoxin-producing strains were not deposited and are available only upon request, limiting reproduction
- LC-MS acquisition parameters were occasionally adjusted to target major unnatural isotope ions and were not fully standardized across strains

## Evidence
