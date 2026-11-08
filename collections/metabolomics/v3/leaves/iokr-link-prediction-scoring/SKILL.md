---
name: iokr-link-prediction-scoring
description: Use when ranking candidate GCF-to-molecular-family links in NPLinker
  output and you want to combine the standardised strain correlation score with IOKR
  scores to improve the rank of validated links.
when_to_use_negative:
- You have no validated (ground-truth) GCF–molecular-family links to evaluate ranks
  against, so the priority-ranking comparison cannot be judged.
- Your input is not GNPS molecular networking data paired with antiSMASH/BigScape
  GCFs and a strain_mappings.json — NPLinker requires this data layout.
- You need a single pre-existing score rather than a combination; the metcalf correlation
  score alone or IOKR alone may suffice and is simpler to compute.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework that loads the genomics/metabolomics data, computes metcalf links
    via npl.get_links, and hosts the combined scoring of GCF–molecular-family links
  repo: https://github.com/NPLinker/nplinker
- name: BigScape
  role: Clusters antiSMASH BGC data into GCFs used as the genomic side of the scored
    links
- name: AntiSMASH
  role: Provides the BGC data organised in the antismash directory as input to GCF
    clustering and linking
- name: GNPS
  role: Source of molecular networking data (spectra.mgf, molecular families, annotations)
    required as metabolomics input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:f04ee6131c3467cdb1c66c98c61eaf84ae208de6ba9b10a56de7d4a8413c3626
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# iokr-link-prediction-scoring

## Summary

Score GCF–molecular-family links with Input-Output Kernel Regression (IOKR), standardise these alongside strain-correlation (metcalf) scores, and combine them via sign-weighted ℓp sums (ℓ1, ℓ2, ℓ1.5) to rank candidate links, so that validated GCF–metabolite links are prioritised better than by either score alone.

## When to use

Use when ranking candidate GCF-to-molecular-family links in NPLinker output and you want to combine the standardised strain correlation score with IOKR scores to improve the rank of validated links.

## When NOT to use

- You have no validated (ground-truth) GCF–molecular-family links to evaluate ranks against, so the priority-ranking comparison cannot be judged.
- Your input is not GNPS molecular networking data paired with antiSMASH/BigScape GCFs and a strain_mappings.json — NPLinker requires this data layout.
- You need a single pre-existing score rather than a combination; the metcalf correlation score alone or IOKR alone may suffice and is simpler to compute.

## Inputs

- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv)
- antiSMASH BGC data clustered into BigScape GCFs
- strain_mappings.json mapping BGCs and spectra to strains
- MIBiG BGC metadata for validated homologous BGCs (e.g. BGC0001228, BGC0000241)
- Standardised strain correlation (metcalf) scores for GCF–molecular-family pairs

## Outputs

- IOKR scores for all GCF–molecular-family pairs
- Combined ℓ1, ℓ2 and ℓ1.5 scores per GCF–molecular-family pair
- Rank counts and best ranks per validated link (table4_link_ranks.csv, layout of Table 4)

## How to apply

Load the Crüsemann data set (GNPS spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv, and antiSMASH-derived BigScape GCFs) into NPLinker via npl.load_data(), using strain_mappings.json to connect BGCs and spectra to strains. Identify the GCFs homologous to the validated MIBiG BGCs (e.g. BGC0001228 for retimycin A, BGC0000241 for lomaiviticin A) and compute links against all molecular families with npl.get_links using the metcalf scoring method as the base correlation score. Standardise the correlation scores and compute IOKR scores for all GCF–molecular-family pairs, then combine the standardised scores into ℓ1, ℓ2 and ℓ1.5 combined scores via the Eq 2 sign-weighted sum. For each validated link, count candidate links whose combined score is equal to or higher than the validated link's score and record the best rank achieved under the ℓ1.5 combination, compiling counts and best ranks into table4_link_ranks.csv in the layout of Table 4. Rationale: using multiple complementary link-scoring functions together makes it easier to prioritise true links relative to others, and the ℓ1.5 combined score assigns the best rank to the validated link in 10 out of 15 cases.

## Related tools

- **NPLinker** (Framework that loads the genomics/metabolomics data, computes metcalf links via npl.get_links, and hosts the combined scoring of GCF–molecular-family links) — https://github.com/NPLinker/nplinker
- **BigScape** (Clusters antiSMASH BGC data into GCFs used as the genomic side of the scored links)
- **AntiSMASH** (Provides the BGC data organised in the antismash directory as input to GCF clustering and linking)
- **GNPS** (Source of molecular networking data (spectra.mgf, molecular families, annotations) required as metabolomics input)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- The ℓ1.5 combined score assigns the best rank to the validated link in 10 out of 15 cases (as reported in Table 4).
- Spot-check known validated links: BGC0001228 (retimycin A) ranked at number 253 and BGC0000241 (lomaiviticin A) at number 5 under the combined score, better than either score alone.
- For every validated link, verify the recorded rank equals the count of candidate links whose combined score is equal to or higher than the validated link's score.
- Confirm output CSV table4_link_ranks.csv matches the column/row layout of Table 4 (counts and best ranks per MIBiG BGC).
- Confirm all correlation scores were standardised before combination, and that ℓ1, ℓ2 and ℓ1.5 combinations use the Eq 2 sign-weighted sum.

## Limitations

- Ranks are evaluated on a limited set of validated links (15 cases in Table 4), so performance gains are demonstrated on this subset rather than exhaustively.
- The combined-score advantage depends on the quality of the base metcalf correlation scores and strain mappings; mis-mapped strains degrade all component scores.
- Ranks can be large even with the combined score (e.g. rank 253 for BGC0001228), so prioritisation improvement is relative, not absolute.

## Evidence

- [abstract] We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others.: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [other] combine the standardised scores into ℓ1, ℓ2, and ℓ1.5 combined scores via the Eq 2 sign-weighted sum: [evidence span withheld — anchor status: not_found]
- [methods] compute the links for the first 3 GCFs using metcalf scoring method: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] NPLinker requires GNPS molecular networking data as input: "NPLinker requires GNPS molecular networking data as input."
- [methods] The strain mappings file strain_mapping.json is required by NPLinker to map the strain to genomics and metabolomics data.: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
