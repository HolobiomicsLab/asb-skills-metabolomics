---
name: spectral-library-matching
description: Use when acquired LC-MS/MS spectra need metabolite annotations (MSI Level
  2/3), or when a query MS/MS spectrum must be searched against public metabolomics
  repositories via tools like GNPS MASST or tissueMASST.
when_to_use_negative:
- When a Level 1 (fully validated structure) annotation is required — spectral matching
  against public libraries only reaches MSI Level 2 or 3.
- When the query features exist in formats not representable as MS/MS peak lists,
  or when tissue-level biological interpretation is the goal without first having
  spectral matches.
- When lack of detection in reference datasets could be misread as biological absence
  — feature detection depends on study-specific extraction methods, workflows, and
  instrumentation.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: GNPS
  role: Provides the public reference spectral libraries against which experimental
    MS/MS spectra are matched for annotation (MSI Level 2/3) and molecular networking.
- name: tissueMASST
  role: Mass spectrometry search tool that queries individual MS/MS spectra against
    44,992 public MS/MS files from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench
    to find tissue- and disease-associated matches.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:ad16c49285d93827d18d075c55b4b998eba88ee93a5856267b3e078e084cdbac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-library-matching

## Summary

Matches experimental tandem MS/MS spectra against curated reference spectral libraries to transfer confident structural annotations onto acquired features, enabling downstream molecular networking, cross-tissue comparisons, and cross-repository search.

## When to use

Use when acquired LC-MS/MS spectra need metabolite annotations (MSI Level 2/3), or when a query MS/MS spectrum must be searched against public metabolomics repositories via tools like GNPS MASST or tissueMASST.

## When NOT to use

- When a Level 1 (fully validated structure) annotation is required — spectral matching against public libraries only reaches MSI Level 2 or 3.
- When the query features exist in formats not representable as MS/MS peak lists, or when tissue-level biological interpretation is the goal without first having spectral matches.
- When lack of detection in reference datasets could be misread as biological absence — feature detection depends on study-specific extraction methods, workflows, and instrumentation.

## Inputs

- Acquired MS/MS peak lists or spectra files (mzML / MGF) from untargeted metabolomics runs
- Query MS/MS spectra with precursor m/z (e.g. benzoyl-, phenylacetyl-, phenylpropionyl-carnitine)
- Public reference spectral libraries (GNPS reference libraries)
- Public MS/MS reference datasets indexed by precursor m/z and annotated with tissue/organism/disease metadata

## Outputs

- Per-feature metabolite annotations at MSI Level 2 or 3 confidence
- Molecular-network nodes linking related spectra through library matches and spectral similarity
- Per-query search-result tables listing matched reference files with source study, organism, tissue, and disease metadata (e.g. tissue_masst_search_results.csv)

## How to apply

Collect MS/MS peak lists from acquired files (mzML/mgf) and match them by cosine similarity of fragment peaks against a reference library such as the public GNPS spectral libraries, which yields Level 2 or 3 Metabolomics Standards Initiative annotations rather than fully validated structures. Use the annotated spectra as nodes in molecular networking by linking experimental spectra to library hits and to each other, while accepting that some pairs may be missing due to poor fragmentation or missing fragments during acquisition. For cross-repository searches, tissueMASST is the companion tool: its reference database comprises 44,992 public MS/MS files as of April 2025 (27,954 human, 11,422 mouse, 1,328 rat) collected from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench. Index reference MS/MS peak lists by precursor m/z, score query spectra (e.g., benzoyl-carnitine at m/z 266.138, phenylacetyl-carnitine at m/z 280.153, phenylpropionyl-carnitine at m/z 294.169) against reference spectra with fragment-peak cosine similarity, and aggregate per-query matches with source study, organism, tissue, and disease-status metadata (58 tissues, >20 diseases such as AD, T2D, IBD). Validate the implementation by confirming that phenylacetyl-carnitine and phenylpropionyl-carnitine queries return matches mapping to human blood and urine, with hit distributions reflecting known tissue and disease enrichment patterns such as PAC enrichment in inflammatory-condition datasets.

## Related tools

- **GNPS** (Provides the public reference spectral libraries against which experimental MS/MS spectra are matched for annotation (MSI Level 2/3) and molecular networking.)
- **tissueMASST** (Mass spectrometry search tool that queries individual MS/MS spectra against 44,992 public MS/MS files from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench to find tissue- and disease-associated matches.)

## Examples

```
python tissue_masst_search.py --query PAC --output tissue_masst_search_results.csv
```

## Evaluation signals

- Query spectra of phenylacetyl-carnitine (m/z 280.153) and phenylpropionyl-carnitine (m/z 294.169) return hits mapping to human blood and urine.
- Hit distributions for phenylacetyl-carnitine reflect enrichment in inflammatory-condition datasets, consistent with Figure 6B.
- Reference database composition matches the paper's stated totals: 44,992 public MS/MS files (27,954 human, 11,422 mouse, 1,328 rat).
- Output file tissue_masst_search_results.csv contains per-query rows with source study, organism, tissue, and disease-status metadata.
- Annotations are flagged as MSI Level 2 or 3, never Level 1, when produced purely by spectral matching.

## Limitations

- Spectral library annotations are Level 2 or 3 per the Metabolomics Standards Initiative, not fully validated structures.
- Some spectral pairs may be absent from the molecular network due to poor fragmentation or missing fragments during MS/MS acquisition.
- Feature detection across tissueMASST reference datasets is confounded by study-specific extraction methods, workflows, and instrumentation; absence of detection does not mean absence in tissue.
- Whether conjugation of microbial phenylalanine-derived precursors into carnitines is microbe- or host-mediated remains unclear, limiting biological interpretation of matching results.

## Evidence

- [discussion] MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation"
- [other] tissueMASST's reference database comprises 44,992 public MS/MS files as of April 2025, of which 27,954 were acquired from human tissues, 11,422 from mice, and 1,328 from rats.: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-tissue molecular networking is based on MS/MS spectral matching: "Cross-tissue molecular networking is based on MS/MS spectral matching"
- [discussion] Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
- [intro] we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
