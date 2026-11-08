---
name: randomised-baseline-auc-evaluation
description: Use when a newly trained link-scoring function (e.g. IOKR on paired MIBiG/GNPS
  BGC-spectrum data) must be compared to a randomised baseline to quantify whether
  it ranks true links above chance, especially at low top-n.
when_to_use_negative:
- When evaluating NPLinker's metcalf scoring or other spectral-link scoring functions
  rather than the IOKR metabolite-to-BGC ranking task — use npl.get_links with the
  appropriate scoring method instead
- When no paired BGC-spectrum ground-truth links are available to permute for the
  null baseline, since the randomised control depends on genuine pairings
- When the goal is de novo structure annotation rather than ranking known MIBiG-associated
  candidate BGC structures against a metabolite
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0092
tools:
- name: NPLinker
  role: Framework that loads the paired MIBiG/GNPS data in local mode, provides the
    IOKR scoring implementation, and hosts the output directory for the performance
    CSV
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of the molecular networking input data (spectra.mgf, molecular_families.tsv,
    annotations.tsv) from GNPS1 or GNPS2 workflows
- name: Python
  role: Runtime environment (>=3.11) in which the IOKR model is trained and the top-n/AUC
    evaluation is computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:442d42e4255527fa870eb741f82c80f3814074e7b7d3fdd640eec152d0bad903
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - NPLinker
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# randomised-baseline-auc-evaluation

## Summary

Benchmarks the IOKR (Input-Output Kernel Regression) BGC–spectrum link scoring function against a randomised-link null baseline by computing top-n ranking hit rates and AUC over the paired MIBiG/GNPS data, reproducing the paper's Table 3 comparison (AUC 0.6534 vs 0.5209).

## When to use

Use when a newly trained link-scoring function (e.g. IOKR on paired MIBiG/GNPS BGC-spectrum data) must be compared to a randomised baseline to quantify whether it ranks true links above chance, especially at low top-n.

## When NOT to use

- When evaluating NPLinker's metcalf scoring or other spectral-link scoring functions rather than the IOKR metabolite-to-BGC ranking task — use npl.get_links with the appropriate scoring method instead
- When no paired BGC-spectrum ground-truth links are available to permute for the null baseline, since the randomised control depends on genuine pairings
- When the goal is de novo structure annotation rather than ranking known MIBiG-associated candidate BGC structures against a metabolite

## Inputs

- Paired MIBiG/GNPS BGC-spectrum dataset (NPLinker local mode example, e.g. nplinker_local_mode_example.zip from Zenodo)
- mibig_json_3.1.tar.gz MIBiG metadata archive (auto-managed by NPLinker)
- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings) from a GNPS1 or GNPS2 workflow
- strain_mappings.json mapping BGC identifiers to spectrum identifiers

## Outputs

- iokr_table3_performance.csv with top-n ranking hit rates and AUC values in the NPLinker output directory
- Top-n hit-rate curve for the IOKR score versus the randomised baseline
- AUC point estimates (IOKR ~0.6534 vs randomised ~0.5209 on the paired MIBiG/GNPS dataset)

## How to apply

Load the paired MIBiG/GNPS dataset with NPLinker in local mode, using strain_mappings.json to associate BGC identifiers (e.g. xxxx.region001 from the antismash directory) with spectrum scan numbers from spectra.mgf. Train the Input-Output Kernel Regression model on the paired set, learning a kernel mapping from mass spectral features to BGC classes, then for each metabolite rank all MIBiG-associated candidate BGC structures using the metabolite-to-BGC rank propagation rule. Generate a randomised-link negative control by permuting the BGC-spectrum pairings so any apparent ranking performance reflects the null distribution. Compute top-n ranking hit rates and the area under the curve (AUC) for both the IOKR score and the randomised baseline, and write the metrics to iokr_table3_performance.csv in the NPLinker output directory. The skill is applied correctly when IOKR outperforms the null by a considerable margin at low top-n, matching the reported 0.6534 vs 0.5209 AUC.

## Related tools

- **NPLinker** (Framework that loads the paired MIBiG/GNPS data in local mode, provides the IOKR scoring implementation, and hosts the output directory for the performance CSV) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of the molecular networking input data (spectra.mgf, molecular_families.tsv, annotations.tsv) from GNPS1 or GNPS2 workflows)
- **Python** (Runtime environment (>=3.11) in which the IOKR model is trained and the top-n/AUC evaluation is computed)

## Evaluation signals

- IOKR AUC on the paired MIBiG/GNPS data should be ~0.6534, clearly exceeding the randomised baseline AUC of ~0.5209
- The IOKR-vs-null gap should be largest at low top-n values (e.g. top-1/top-5 hit rates well above the permuted baseline)
- The randomised baseline must be produced by permuting BGC-spectrum pairings on the same candidate set (all MIBiG-associated structures), not by sampling different candidates
- iokr_table3_performance.csv exists in the NPLinker output directory and contains per-n hit rates plus AUC for both score and null
- Every metabolite is ranked against the full MIBiG candidate set using the metabolite-to-BGC rank propagation rule, not a filtered subset

## Limitations

- Evaluation is restricted to MIBiG-associated structures as the candidate set, so performance does not generalise to unknown BGCs outside MIBiG
- The randomised baseline is a null distribution of permuted pairings; it controls for chance ranking but not for other confounders such as GCF size or spectrum quality
- NPLinker currently accepts GNPS molecular networking data only from GNPS1 (gnps.ucsd.edu) and GNPS2 (gnps2.org) workflows, and requires AntiSMASH BGC data as genomics input

## Evidence

- [other] IOKR outperforms the randomised baseline by a considerable margin, especially at low top-n values, achieving an AUC of 0.6534 versus 0.5209 for the null distribution.: [evidence span withheld — anchor status: not_found]
- [abstract] introduce a novel score using an Input-Output Kernel Regression approach: "introduce a novel score using an Input-Output Kernel Regression approach"
- [abstract] Results are verified using publicly available data sets that include validated links.: "Results are verified using publicly available data sets that include validated links."
- [methods] NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
