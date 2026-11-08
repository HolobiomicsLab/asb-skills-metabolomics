---
name: bgc-spectrum-link-ranking
description: Use when paired genomic-metabolomic data (AntiSMASH BGCs, GNPS molecular
  networking spectra, strain mappings) must be linked and candidate links prioritised
  by rank, e.g. to reproduce Table 3-style top-n performance comparisons.
when_to_use_negative:
- Input omics data lack the strain mapping file (strain_mapping.json) required to
  associate genomic and metabolomic datasets, so links cannot be validated against
  known strains
- The goal is raw GNPS molecular networking or de novo spectrum clustering rather
  than ranking candidate BGC links
- No candidate structure set (e.g. MIBiG-associated structures) is available, so top-n
  ranking against candidates is not meaningful
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework that loads the paired genomics/metabolomics data, computes the link
    graph between GCFs and spectra, and exposes scoring functions including metcalf
    and Input-Output Kernel Regression
  repo: https://github.com/NPLinker/nplinker
- name: AntiSMASH
  role: Source of the BGC data (xxxx.region001 identifiers) used as the ranked candidates
- name: BigScape
  role: Clusters AntiSMASH BGCs into gene cluster families (GCFs) that NPLinker runs
    automatically if the bigscape directory is absent; both version 1 and 2 supported
- name: GNPS
  role: Provides the molecular networking data (spectra.mgf, molecular_families.tsv,
    annotations.tsv) whose spectra are ranked against BGC candidates
- name: Python
  role: Runtime for the NPLinker pipeline (Python >=3.11 required)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:46f218cccb626234f91e88edc12fef80c01a99e287965ce9ee51d16786dd34b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bgc-spectrum-link-ranking

## Summary

Ranks candidate BGC (gene cluster) structures against each metabolite's spectrum using scoring functions such as the standardised Metcalf score and Input-Output Kernel Regression (IOKR), then evaluates top-n hit rates and AUC against a randomised-link baseline within the NPLinker framework.

## When to use

Use when paired genomic-metabolomic data (AntiSMASH BGCs, GNPS molecular networking spectra, strain mappings) must be linked and candidate links prioritised by rank, e.g. to reproduce Table 3-style top-n performance comparisons.

## When NOT to use

- Input omics data lack the strain mapping file (strain_mapping.json) required to associate genomic and metabolomic datasets, so links cannot be validated against known strains
- The goal is raw GNPS molecular networking or de novo spectrum clustering rather than ranking candidate BGC links
- No candidate structure set (e.g. MIBiG-associated structures) is available, so top-n ranking against candidates is not meaningful

## Inputs

- AntiSMASH BGC data in the antismash subdirectory of the NPLinker working directory
- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings) from GNPS1 or GNPS2 workflows
- MIBiG metadata archive (mibig_json_3.1.tar.gz, auto-downloaded by NPLinker)
- strain_mappings.json mapping BGC identifiers to spectrum identifiers
- Paired Omics Data Platform (PODP) dataset as NPLinker input

## Outputs

- Ranked list of MIBiG-associated candidate BGC structures per metabolite
- Top-n ranking hit rates for the scoring function and the randomised baseline
- AUC comparison (e.g. iokr_table3_performance.csv in the NPLinker output directory)
- Link graph object (e.g. from npl.get_links)

## How to apply

Load the paired dataset with NPLinker in local mode, using strain_mappings.json to associate BGC identifiers (e.g. xxxx.region001 from the antismash directory) with spectrum identifiers (scan numbers from spectra.mgf). Compute a link graph between GCFs and spectra with a scoring method (e.g. 'metcalf'), or train an Input-Output Kernel Regression model on the paired MIBiG-GNPS BGC-spectrum set to learn a kernel mapping from mass spectral features to BGC classes. For each metabolite, rank all MIBiG-associated candidate BGC structures using the metabolite-to-BGC rank propagation rule. Generate a randomised-link negative control by permuting the BGC-spectrum pairings, then compute top-n ranking hit rates and AUC for the scoring function versus the null distribution (IOKR AUC 0.6534 vs 0.5209 randomised in Table 3).

## Related tools

- **NPLinker** (Framework that loads the paired genomics/metabolomics data, computes the link graph between GCFs and spectra, and exposes scoring functions including metcalf and Input-Output Kernel Regression) — https://github.com/NPLinker/nplinker
- **AntiSMASH** (Source of the BGC data (xxxx.region001 identifiers) used as the ranked candidates)
- **BigScape** (Clusters AntiSMASH BGCs into gene cluster families (GCFs) that NPLinker runs automatically if the bigscape directory is absent; both version 1 and 2 supported)
- **GNPS** (Provides the molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv) whose spectra are ranked against BGC candidates)
- **Python** (Runtime for the NPLinker pipeline (Python >=3.11 required))

## Examples

```
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- IOKR ranking AUC should exceed the randomised baseline by a clear margin, especially at low top-n values (reported: 0.6534 IOKR vs 0.5209 for the null distribution)
- Ranking metrics are written to iokr_table3_performance.csv in the NPLinker output directory and reproduce Table 3's comparison
- Every ranked metabolite-to-BGC link resolves through the strain_mappings.json associations (BGC identifiers like xxxx.region001 matched to scan numbers from spectra.mgf)
- The randomised negative control is produced by permuting BGC-spectrum pairings and its AUC should fall near the expected null value (~0.52)
- The link graph is computed for the intended GCF subset, e.g. npl.get_links(npl.gcfs[:3], 'metcalf')

## Limitations

- Performance is quantified only against the MIBiG-associated candidate structure set; links outside this candidate set are not evaluated
- The randomised-link baseline is a permuted-pairing null, so absolute AUC values reflect the candidate set composition and paired dataset used
- The discussion section in the retrieved context contains only a dependency bump changelog, giving no discussion of limitations or reproducibility gaps

## Evidence

- [results] iokr-vs-baseline finding: [evidence span withheld — anchor status: not_found]
- [methods] link computation API: "# compute the links for the first 3 GCFs using metcalf scoring method
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] GNPS input requirement: "NPLinker requires GNPS molecular networking data as input. It currently accepts data from both GNPS1 (https://gnps.ucsd.edu) and GNPS2 (https://gnps2.org) workflows."
- [abstract] complementary scoring rationale: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [methods] MIBiG auto-download: "`mibig` directory contains the MIBiG metadata, which is automatically created and downloaded by NPLinker"
