---
name: casmi-contest-score-retrieval-and-harmonization
description: 'Use when benchmarking an annotation confidence score against published
  contest scores: you need the CASMI 2016 per-compound hit scores from MetFrag, MAGMa+,
  CFM-ID, CSI:FingerID and CSI:FingerID IOKR to compute ROC curves, FDR-corrected
  hit counts, and mean ranks of correct hits.'
when_to_use_negative:
- You only want to re-run the annotation tools on raw mzML/mzXML spectra instead of
  comparing the published contest scores — re-running will not reproduce the exact
  CASMI 2016 submissions.
- The annotation confidence score being evaluated was not computed on the same 127
  CASMI 2016 compounds or the same candidate databases, so score comparability breaks.
- The task is spectral library search de-replication on biological data, not contest-score
  retrieval for method benchmarking on the CASMI 2016 reference compounds.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetFrag
  role: Contest baseline score whose per-compound hit scores are downloaded from the
    CASMI 2016 results web page for benchmark comparison
- name: MAGMa+
  role: Contest baseline score retrieved from the CASMI 2016 results web page
- name: CFM-ID
  role: Contest baseline score retrieved from the CASMI 2016 results web page
- name: CSI:FingerID (original)
  role: Contest baseline score retrieved from the CASMI 2016 results web page
- name: CSI:FingerID IOKR
  role: Contest baseline score retrieved from the CASMI 2016 results web page
- name: COSMIC
  role: Confidence score to be benchmarked against the CASMI 2016 contest scores on
    the 127 positive-ion Orbitrap standard compounds
  repo: github.com/sirius-ms/sirius
- name: CSI:FingerID 1.2.0
  role: Used to compute scores for the structure–disjoint evaluation of CSI:FingerID
    when harmonizing candidate lists
  repo: github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:8a5de7d2b8d8ad098f6ee3e28849db7f2893b32a6e90290659c1b58592c0b791
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID (original)
  - CSI:FingerID 1.2.0
  - CSI:FingerID IOKR
  - MAGMa+
  - MetFrag
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# casmi-contest-score-retrieval-and-harmonization

## Summary

Retrieve the published per-compound hit scores of the CASMI 2016 contest methods (MetFrag, MAGMa+, CFM-ID, CSI:FingerID original and CSI:FingerID IOKR) from the contest results web page, merge them with COSMIC confidence scores for the same 127 CASMI 2016 compounds, and harmonize the candidate lists so that every score is evaluated on an identical ranking task.

## When to use

Use when benchmarking an annotation confidence score against published contest scores: you need the CASMI 2016 per-compound hit scores from MetFrag, MAGMa+, CFM-ID, CSI:FingerID and CSI:FingerID IOKR to compute ROC curves, FDR-corrected hit counts, and mean ranks of correct hits.

## When NOT to use

- You only want to re-run the annotation tools on raw mzML/mzXML spectra instead of comparing the published contest scores — re-running will not reproduce the exact CASMI 2016 submissions.
- The annotation confidence score being evaluated was not computed on the same 127 CASMI 2016 compounds or the same candidate databases, so score comparability breaks.
- The task is spectral library search de-replication on biological data, not contest-score retrieval for method benchmarking on the CASMI 2016 reference compounds.

## Inputs

- Published per-compound hit scores from the CASMI 2016 results web page for MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR
- COSMIC confidence scores for the 127 positive-ion Orbitrap CASMI 2016 standard compounds
- Known correct structure for each CASMI 2016 compound
- Candidate structure lists per compound (e.g. biomolecule structure database, ChemSpider) with molecular formulas

## Outputs

- Harmonized per-compound score tables with one ranking per CASMI 2016 compound per method
- casmi2016_roc_hop_results.csv with AUC, correct-hit counts at 10% and 0% FDR, and mean rank of correct hits per method
- ROC curve and hop plot figures (PNG) per score

## How to apply

Download the per-compound hit scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR directly from the CASMI 2016 results web page rather than re-running the tools, so the exact contest submission scores are compared. Load the COSMIC confidence scores computed for the 127 positive-ion Orbitrap CASMI 2016 standard compounds, and for each score per compound rank the candidate structures against the known correct structure. Restrict the candidate list to candidates sharing the molecular formula of the top-scoring hit where the score requires a formula-consistent set, matching the article's structure–disjoint evaluation setup (CSI:FingerID 1.2.0 was used for the structure–disjoint evaluation of CSI:FingerID scores). Write the harmonized per-compound score tables and compute the baseline targets: area under the ROC curve of 0.40–0.55 for the contest methods versus ~0.82 for COSMIC, 57 correct hits at FDR below 10% with the biomolecule structure database and 16 correct hits at 0% FDR with ChemSpider, and mean rank improvement of the correct structure from 56.5 to 33.9.

## Related tools

- **MetFrag** (Contest baseline score whose per-compound hit scores are downloaded from the CASMI 2016 results web page for benchmark comparison)
- **MAGMa+** (Contest baseline score retrieved from the CASMI 2016 results web page)
- **CFM-ID** (Contest baseline score retrieved from the CASMI 2016 results web page)
- **CSI:FingerID (original)** (Contest baseline score retrieved from the CASMI 2016 results web page)
- **CSI:FingerID IOKR** (Contest baseline score retrieved from the CASMI 2016 results web page)
- **COSMIC** (Confidence score to be benchmarked against the CASMI 2016 contest scores on the 127 positive-ion Orbitrap standard compounds) — github.com/sirius-ms/sirius
- **CSI:FingerID 1.2.0** (Used to compute scores for the structure–disjoint evaluation of CSI:FingerID when harmonizing candidate lists) — github.com/sirius-ms/sirius

## Examples

```
curl -o casmi2016_scores.csv 'https://www.csi-fingerid.uni-jena.de/casmi2016/'
```

## Evaluation signals

- The contest methods' ROC AUC falls in the expected 0.40–0.55 range while COSMIC's confidence score reaches approximately 0.82.
- COSMIC correctly annotates 57 hits with FDR below 10% using the biomolecule structure database (123 queries) and 16 hits with FDR 0% using ChemSpider (127 queries).
- The mean rank of the correct structure improves from 56.5 (contest scores) to 33.9 (COSMIC confidence score).
- The per-compound candidate lists are restricted to candidates with the same molecular formula as the top-scoring hit so scores are compared on an identical ranking task.
- All metrics are written to casmi2016_roc_hop_results.csv and ROC/hop plots are exported as PNG figures.

## Limitations

- Scores must be downloaded from the CASMI 2016 results web page rather than recomputed, since re-running the tools will not reproduce the exact contest submissions.
- The harmonized comparison requires that the COSMIC confidence scores were computed on the same 127 positive-ion Orbitrap standard compounds and the same candidate structure databases (biomolecule structure database or ChemSpider) as the contest scores.
- CASMI 2016 evaluations rely on reference measurements with known correct answers; the same benchmark cannot be reproduced for biological datasets where the ground truth is unknown.

## Evidence

- [discussion] CASMI 2016 contest baseline scores are fetched from the results web page, not re-run: "Scores of MetFrag, MAGMa+, CFM-ID, CSI:FingerID (original) and CSI:FingerID IOKR were downloaded from the CASMI 2016 results web page"
- [discussion] Structure–disjoint evaluation of CSI:FingerID used CSI:FingerID 1.2.0: "We computed scores for the structure–disjoint evaluation of CSI:FingerID using CSI:FingerID 1.2.0."
- [discussion] Candidate list harmonization by formula-consistent restriction: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [abstract] Expected quantitative targets for the harmonized comparison: "COSMIC annotates a substantial number of hits at low false discovery rates and outperforms spectral library search"
- [discussion] SIRIUS processes the LC–MS/MS runs from which COSMIC scores are computed: "SIRIUS 4 was used to process LC–MS/MS runs and MassIVE datasets provided in mzML or mzXML format."
- [methods] SIRIUS releases from which the score-computation tooling is obtained: "All (including previous) releases can be found [here](https://github.com/sirius-ms/sirius/releases)."
