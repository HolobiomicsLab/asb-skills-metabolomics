---
name: input-output-kernel-regression-scoring
description: Use when GNPS molecular networking spectra must be linked to candidate
  biosynthetic gene clusters with a supervised kernel-regression score, or when reproducing
  the IOKR top-n ranking and AUC benchmarks (Table 3) against a randomised baseline.
when_to_use_negative:
- The dataset has no paired BGC-spectrum training links (e.g. no PODP or strain_mappings.json
  ground truth), so the kernel mapping from mass spectral features to BGC classes
  cannot be learned
- The input is raw MS data that has not been processed into GNPS molecular networking
  output (spectra.mgf plus molecular_families.tsv, annotations.tsv, file_mappings)
- The goal is raw Metcalf-style correlation scoring alone rather than the supervised
  IOKR ranking of candidate MIBiG structures
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Python framework that integrates the genomics and metabolomics data, loads
    the paired MIBiG/GNPS dataset in local mode, trains the IOKR scoring model and
    computes the link graphs
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of the molecular networking data (spectra.mgf, molecular_families.tsv,
    annotations.tsv, file_mappings) accepted by NPLinker from GNPS1 and GNPS2 workflows
  repo: https://gnps.ucsd.edu
- name: Python
  role: Runtime (version >= 3.11) in which NPLinker runs the IOKR training, ranking
    and significance-test scripts
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:e3cf385bbb765017c876fc019e49ff5973a2dda87f11827d417bccc92c2ed417
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# input-output-kernel-regression-scoring

## Summary

Scores metabolite-to-BGC links using Input-Output Kernel Regression (IOKR), trained on paired MIBiG/GNPS BGC-spectrum data, and ranks all MIBiG-associated candidate structures per spectrum. Within the NPLinker framework this IOKR score is applied alongside the standardised strain correlation score to prioritise validated genomic-metabolomic links.

## When to use

Use when GNPS molecular networking spectra must be linked to candidate biosynthetic gene clusters with a supervised kernel-regression score, or when reproducing the IOKR top-n ranking and AUC benchmarks (Table 3) against a randomised baseline.

## When NOT to use

- The dataset has no paired BGC-spectrum training links (e.g. no PODP or strain_mappings.json ground truth), so the kernel mapping from mass spectral features to BGC classes cannot be learned
- The input is raw MS data that has not been processed into GNPS molecular networking output (spectra.mgf plus molecular_families.tsv, annotations.tsv, file_mappings)
- The goal is raw Metcalf-style correlation scoring alone rather than the supervised IOKR ranking of candidate MIBiG structures

## Inputs

- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings) from a GNPS1 or GNPS2 workflow
- MIBiG metadata archive (mibig_json_3.1.tar.gz, auto-downloaded by NPLinker)
- strain_mappings.json file mapping strains to genomics and metabolomics data
- AntiSMASH BGC data organised in the antismash subdirectory (e.g. xxxx.region001 identifiers)
- Validated-link ground-truth labels for the Crüsemann, Gross and Leão data sets (for enrichment analysis)
- Paired Omics Data Platform (PODP) dataset used as input for NPLinker

## Outputs

- Ranked list of MIBiG-associated candidate BGC structures per metabolite (metabolite-to-BGC rank propagation)
- Top-n ranking hit rates and AUC for IOKR versus the randomised baseline (Table 3 comparison: AUC 0.6534 vs 0.5209)
- iokr_table3_performance.csv in the NPLinker output directory
- table2_recomputed.csv with 90th-percentile enrichment proportions and pooled p-values (IOKR p = 0.0139; standardised correlation p = 2.483e-11; joint enrichment p = 2.633e-4 and p = 0.0208)

## How to apply

Download the MIBiG metadata archive (mibig_json_3.1.tar.gz, auto-managed by NPLinker) and GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings) from a supported GNPS1 or GNPS2 workflow. Load the paired dataset with NPLinker in local mode, using strain_mappings.json to associate BGC identifiers (e.g. xxxx.region001 from the antismash directory) with spectrum identifiers (scan numbers from spectra.mgf). Train the Input-Output Kernel Regression model on the paired MIBiG-GNPS BGC-spectrum set, learning a kernel mapping from mass spectral features to BGC classes. For each metabolite, rank all MIBiG-associated candidate BGC structures using the metabolite-to-BGC rank propagation rule, and generate a randomised-link negative control by permuting the BGC-spectrum pairings. Compute top-n ranking hit rates and AUC for the IOKR score versus the randomised baseline (expect AUC 0.6534 vs 0.5209, reproducing Table 3); optionally, apply a 90th-percentile threshold to IOKR and the standardised strain correlation scores jointly, where links exceeding both thresholds are significantly enriched for validated links (p = 2.633e-4 from IOKR, p = 0.0208 from standardised correlation).

## Related tools

- **NPLinker** (Python framework that integrates the genomics and metabolomics data, loads the paired MIBiG/GNPS dataset in local mode, trains the IOKR scoring model and computes the link graphs) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of the molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings) accepted by NPLinker from GNPS1 and GNPS2 workflows) — https://gnps.ucsd.edu
- **Python** (Runtime (version >= 3.11) in which NPLinker runs the IOKR training, ranking and significance-test scripts)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Reproduced IOKR AUC on the paired MIBiG/GNPS data equals 0.6534 versus 0.5209 for the randomised (permuted-pairing) baseline
- IOKR top-n hit rates exceed the randomised baseline by a considerable margin, especially at low top-n values
- table2_recomputed.csv matches reported pooled significance values: IOKR p = 0.0139, standardised correlation p = 2.483e-11, joint enrichment p = 2.633e-4 and p = 0.0208
- Links above the 90th percentile on both IOKR and standardised strain correlation scores show higher proportions of validated links than either single score alone
- iokr_table3_performance.csv is written to the NPLinker output directory with the Table 3 ranking metrics

## Limitations

- Requires GNPS molecular networking data as input; NPLinker currently accepts data only from GNPS1 (https://gnps.ucsd.edu) and GNPS2 (https://gnps2.org) workflows
- The negative control is a randomised-link permutation of BGC-spectrum pairings, so the AUC comparison is against a null distribution rather than an alternative scoring method
- Candidate ranking is restricted to the MIBiG-associated structure set; metabolites without a MIBiG-associated candidate BGC cannot be meaningfully ranked

## Evidence

- [abstract] A novel score based on Input-Output Kernel Regression is introduced.: "introduce a novel score using an Input-Output Kernel Regression approach"
- [abstract] Multiple scoring functions prioritise true links.: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [methods] NPLinker integrates genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [methods] GNPS data sources for the IOKR inputs.: "It currently accepts data from both GNPS1 (https://gnps.ucsd.edu) and GNPS2 (https://gnps2.org) workflows."
- [methods] Strain mapping file associates BGC and spectrum identifiers.: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [methods] NPLinker computes link graphs with selectable scoring methods.: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
