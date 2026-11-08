---
name: biomarker-candidate-selection
description: Use when a peak-aligned, TIC-normalized SCMS metabolomics feature table
  (e.g., primary vs metastatic melanoma cell pairs, control and vemurafenib-treated)
  must be screened for species that differ between groups and annotated as biomarker
  candidates.
when_to_use_negative:
- The goal is cell phenotype classification rather than building a differential-species
  biomarker list — use MetaPhenotype (MAML meta-learning) instead, which was built
  to classify primary vs metastatic cells from limited SCMS samples
- The data are raw SCMS spectra rather than a pretreated, peak-aligned, TIC-normalized
  feature table — the established pretreatment protocol (noise/background subtraction,
  Geena2 10 ppm alignment) must run first
- Only a handful of cells per group are available for the new comparison — statistical
  fold-change/FDR screening is unreliable at very low cell counts, which is the regime
  MetaPhenotype's few-shot adaptation targets instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalyst 6.0
  role: Render volcano plots of log2 fold change versus -log10 FDR-adjusted p-value
    (also used for PCA on the same SCMS feature table)
- name: Geena2
  role: Online bioinformatics tool used for peak alignment of SCMS features with a
    mass tolerance of 10 ppm, producing the feature table this selection runs on
- name: METLIN
  role: Online database used to tentatively label/annotate the detected significant
    species by lipid class
- name: HMDB
  role: Online database used alongside METLIN to tentatively label the detected species
- name: MetaPhenotype
  role: Complementary meta-learning model that identifies the small number of essential
    species critical for primary vs metastatic classification, cross-checking biomarker
    candidates from the volcano analysis
  repo: https://github.com/songyuan93/MetaPhenotype
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:4e7c36d2d97ad2ff255837ffff9308052554086f9988cf3bf0b157012408cd0b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/songyuan93/MetaPhenotype
  tools:
  - Geena2
  - HMDB
  - METLIN
  - MetaPhenotype
  - MetaboAnalyst 6.0
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# biomarker-candidate-selection

## Summary

Selects differentially abundant metabolite/lipid species from single-cell mass spectrometry (SCMS) feature tables as biomarker candidates by combining volcano-plot statistics (fold change ≥ 2.0, FDR-adjusted p ≤ 0.05) with tentative database annotation of significant lipid classes.

## When to use

Use when a peak-aligned, TIC-normalized SCMS metabolomics feature table (e.g., primary vs metastatic melanoma cell pairs, control and vemurafenib-treated) must be screened for species that differ between groups and annotated as biomarker candidates.

## When NOT to use

- The goal is cell phenotype classification rather than building a differential-species biomarker list — use MetaPhenotype (MAML meta-learning) instead, which was built to classify primary vs metastatic cells from limited SCMS samples
- The data are raw SCMS spectra rather than a pretreated, peak-aligned, TIC-normalized feature table — the established pretreatment protocol (noise/background subtraction, Geena2 10 ppm alignment) must run first
- Only a handful of cells per group are available for the new comparison — statistical fold-change/FDR screening is unreliable at very low cell counts, which is the regime MetaPhenotype's few-shot adaptation targets instead

## Inputs

- Peak-aligned, TIC-normalized SCMS metabolomics feature table for primary/metastatic melanoma cell pairs (control and vemurafenib-treated), produced by Geena2 alignment at 10 ppm mass tolerance
- Group labels (primary vs metastatic) for each single cell, per pair and treatment condition

## Outputs

- Volcano plots (log2 fold change vs -log10 FDR-adjusted p-value), one per cell pair/condition, with lipid classes annotated
- Filtered list of significant lipid species meeting fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, with tentative lipid-class labels (TG, Cer, SM, PG, PE) from METLIN/HMDB, reproducing Tables S4/S5

## How to apply

Start from the pretreated SCMS feature table produced by the established protocol (Geena2 peak alignment with a 10 ppm mass tolerance, noise/background subtraction, TIC normalization, missing-value removal). For each cell pair and treatment condition, compute the log2 fold change of each lipid species abundance between primary and metastatic groups and derive FDR-adjusted p-values from the group statistical tests. Filter species to those with fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05, then render one volcano plot per pair/condition (log2 fold change vs -log10 FDR-adjusted p) using MetaboAnalyst 6.0 or an equivalent Python plotting tool, annotating lipid classes. Tentatively assign lipid-class identities of significant species against METLIN and HMDB. The expected outcome is a biomarker candidate table mirroring Tables S4/S5, populated by triglycerides (TG), ceramides (Cer), sphingomyelin (SM), phosphatidylglycerol (PG), and phosphatidylethanolamine (PE).

## Related tools

- **MetaboAnalyst 6.0** (Render volcano plots of log2 fold change versus -log10 FDR-adjusted p-value (also used for PCA on the same SCMS feature table))
- **Geena2** (Online bioinformatics tool used for peak alignment of SCMS features with a mass tolerance of 10 ppm, producing the feature table this selection runs on)
- **METLIN** (Online database used to tentatively label/annotate the detected significant species by lipid class)
- **HMDB** (Online database used alongside METLIN to tentatively label the detected species)
- **MetaPhenotype** (Complementary meta-learning model that identifies the small number of essential species critical for primary vs metastatic classification, cross-checking biomarker candidates from the volcano analysis) — https://github.com/songyuan93/MetaPhenotype

## Evaluation signals

- The significant species sets in each volcano plot visually match the differential species reported in Tables S4/S5, with TG, Cer, SM, PG, and PE members reproduced
- Applying the thresholds fold change ≥ 2.0 and FDR-adjusted p ≤ 0.05 recovers exactly the reported biomarker candidates per cell pair and treatment condition
- Every retained species carries a log2 fold change and an FDR-adjusted p-value computed between primary and metastatic groups for the specific pair/condition
- Tentative lipid-class annotations of the significant species are confirmed against METLIN and HMDB entries
- The input table traces back to the established pretreatment chain (export peaks, noise/background subtraction, TIC normalization, Geena2 10 ppm alignment, missing-value removal)

## Limitations

- Species annotations are only tentative: METLIN and HMDB label detected species without definitive structural confirmation
- PCA on the same SCMS data cannot effectively discriminate phenotypically similar cells (e.g., IGR39/IGR37), so differential statistics rather than unsupervised separation drive candidate selection
- Candidate lists are specific to the analyzed cell pair and vemurafenib treatment condition; transfer to other pairs or treatments requires rerunning the differential analysis per condition

## Evidence

- [intro] Geena2 peak alignment: "an online bioinformatics tool, Geena2,104 for peak alignment (with a mass tolerance of 10 ppm)"
- [intro] Database annotation: "The online databases (METLIN118 and HMDB119) were used to tentatively label the detected species"
- [readme] Input feature table format: [evidence span withheld — anchor status: external_doc]
