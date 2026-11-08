---
name: structure-candidate-ranking
description: Use when untargeted metabolomics spectra (mzML/mzXML, positive ion mode)
  cannot be matched to spectral libraries and you need ranked, confidence-scored structure
  annotations from a structure database such as HMDB or a combinatorial candidate
  set.
when_to_use_negative:
- Reference MS/MS spectra exist in spectral libraries (MassBank, GNPS, NIST 2017)
  — spectral library search is the appropriate first-line de-replication method
- You need stereochemical assignment — the method considers only the two-dimensional
  structure, as stereochemistry cannot be established from fragmentation spectra
- Input is a peptide/protein ID problem or a processed feature table lacking fragmentation
  spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Preprocesses LC–MS/MS runs, merges fragmentation spectra per feature, and
    determines molecular formulas de novo at 10 ppm before candidate ranking
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates against the structure database using default parameters,
    with candidates restricted to the top hit's molecular formula
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Assigns the confidence score (kernel density P value estimation + SVM with
    Platt posterior probabilities) used to filter high-confidence annotations
  repo: https://github.com/sirius-ms/sirius
- name: PySirius / RSirius SDK
  role: Programmatic (Python/R) access to the SIRIUS REST API for scripting the preprocessing
    and ranking pipeline
  repo: https://github.com/sirius-ms/sirius-client-openAPI
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:b7ced17df2377120acda0af296fce1cd2ee77b82ffa0938c9e017bb974446c3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - PySirius / RSirius SDK
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# structure-candidate-ranking

## Summary

Rank molecular structure candidates for untargeted LC–MS/MS features by combining SIRIUS 4 de novo formula assignment, CSI:FingerID structure candidate search, and the COSMIC confidence score, so that in silico annotations absent from spectral libraries can be filtered to high confidence (~FDR 10% at score threshold 0.64).

## When to use

Use when untargeted metabolomics spectra (mzML/mzXML, positive ion mode) cannot be matched to spectral libraries and you need ranked, confidence-scored structure annotations from a structure database such as HMDB or a combinatorial candidate set.

## When NOT to use

- Reference MS/MS spectra exist in spectral libraries (MassBank, GNPS, NIST 2017) — spectral library search is the appropriate first-line de-replication method
- You need stereochemical assignment — the method considers only the two-dimensional structure, as stereochemistry cannot be established from fragmentation spectra
- Input is a peptide/protein ID problem or a processed feature table lacking fragmentation spectra

## Inputs

- Centroided LC–MS/MS runs in .mzML or .mzXML format (positive ion mode; profiled and negative-mode runs discarded)
- MassIVE dataset downloads (e.g. MSV000082973 mice fecal, MSV000080553 rosemary, ten human datasets such as MSV000083559)
- Structure database (e.g. HMDB structure database with 113,983 compounds, combinatorial bile acid conjugate database, biomolecule structure database)
- Reference databases for novelty filtering (HMDB, PubChem)

## Outputs

- Ranked structure annotations per feature with CSI:FingerID scores and COSMIC confidence scores
- High-confidence annotation table filtered at score >= 0.64 with estimated q-values (FDR)
- Novel-annotation tables of unique structures absent from HMDB/spectral libraries (e.g. novel_bile_acid_annotations.tsv, 1,715 Orbitrap annotations)

## How to apply

Preprocess LC–MS/MS runs with SIRIUS 4 (default parameters, 10 ppm mass accuracy): feature detection associating precursor/adduct/isotope peaks, merging of fragmentation spectra per feature by agglomerative clustering on cosine distance, and de novo molecular formula determination using isotope and fragmentation patterns. Rank structure candidates with CSI:FingerID against the chosen structure database (e.g. the 113,983-compound HMDB structure database, a combinatorially generated bile acid conjugate database, or the biomolecule structure database), restricting the candidate list to those with the same molecular formula as the highest-scoring hit and discarding compounds beyond 700 Da to limit running time. Compute the COSMIC confidence score, which maps SVM decision values to posterior probabilities via Platt probabilities and combines kernel density P value estimation with an SVM with enforced feature directionality (SVM C parameter selected by nested cross-validation over {10^-5,...,10^5}). Filter annotations to high confidence at score threshold 0.64 (roughly FDR 10%) and, for novel-annotation counts, exclude hits whose structures are present in HMDB or PubChem.

## Related tools

- **SIRIUS 4** (Preprocesses LC–MS/MS runs, merges fragmentation spectra per feature, and determines molecular formulas de novo at 10 ppm before candidate ranking) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates against the structure database using default parameters, with candidates restricted to the top hit's molecular formula) — https://github.com/sirius-ms/sirius
- **COSMIC** (Assigns the confidence score (kernel density P value estimation + SVM with Platt posterior probabilities) used to filter high-confidence annotations) — https://github.com/sirius-ms/sirius
- **PySirius / RSirius SDK** (Programmatic (Python/R) access to the SIRIUS REST API for scripting the preprocessing and ranking pipeline) — https://github.com/sirius-ms/sirius-client-openAPI

## Examples

```
conda install -c conda-forge py-sirius-ms
```

## Evaluation signals

- Every retained annotation carries a COSMIC confidence score >= 0.64 (threshold roughly corresponding to 10% FDR)
- Each candidate list contains only structures sharing the molecular formula of the top-scoring hit
- No ranked compound exceeds 700 Da and spectra with at most one peak were removed before processing
- Annotation counts match expected cohort values: e.g. 911 high-confidence annotations from 24,554, 315 human structures absent from HMDB, and 1,715 novel Orbitrap annotations after discarding 1,815 training-data structures
- Novel bile acid annotations have estimated q-values below 0.1 and expected hits (Phe-CDCA, Trp-CDCA) appear among the top 12

## Limitations

- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging; the 0.64 threshold is practical guidance only and guarantees FDR in neither direction
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone, and only the two-dimensional structure is considered

## Evidence

- [discussion] CSI:FingerID with default parameters was used to rank structure candidates.: "CSI:FingerID with default parameters was used to rank structure candidates."
- [discussion] Restrict candidates to top hit's formula: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [abstract] COSMIC workflow definition: "Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score"
- [discussion] Mass cutoff for candidate ranking: "Compounds with mass beyond 700 Da were discarded to avoid high running time."
- [readme] SIRIUS fragmentation tree computation: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments"
