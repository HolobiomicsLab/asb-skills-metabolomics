---
name: reference-gene-set-validation
description: Use when a candidate causal gene prioritization tool needs quantitative
  validation against expert-curated mQTL/pQTL reference gene assignments (e.g., Supplementary
  Table S1) to estimate sensitivity and specificity.
when_to_use_negative:
- Input QTLs are trans-pQTLs, since ProGeM's top-down strategy 'would call for a different
  top-down strategy to the one adopted here' and is not directly applicable
- No high-confidence reference causal gene assignments exist for the QTL class in
  question, so sensitivity cannot be estimated without circularity
- The tool and reference set share annotation sources (e.g., KEGG used both by ProGeM
  and the expert review), which would inflate apparent sensitivity
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3053
tools:
- name: ProGeM
  role: R-script framework that prioritizes bottom-up, top-down, and cooccurring candidate
    causal genes at molecular QTLs, whose outputs are compared against the curated
    reference gene set
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Language in which ProGeM is implemented and sourced via ProGeM_settings.R
    to execute the full pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:c27c6c84eb5368bfda021ea91bf1cb4a624a6a2f4179c14d261e88477b7303ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# reference-gene-set-validation

## Summary

Benchmark a causal gene prioritization framework (ProGeM) against a high-confidence reference set of 227 curated metabolite QTLs (and 562 cis-pQTLs), computing sensitivity as recovery of expert-curated causal genes and specificity as true positives per prioritized candidate.

## When to use

Use when a candidate causal gene prioritization tool needs quantitative validation against expert-curated mQTL/pQTL reference gene assignments (e.g., Supplementary Table S1) to estimate sensitivity and specificity.

## When NOT to use

- Input QTLs are trans-pQTLs, since ProGeM's top-down strategy 'would call for a different top-down strategy to the one adopted here' and is not directly applicable
- No high-confidence reference causal gene assignments exist for the QTL class in question, so sensitivity cannot be estimated without circularity
- The tool and reference set share annotation sources (e.g., KEGG used both by ProGeM and the expert review), which would inflate apparent sensitivity

## Inputs

- Tab-separated sentinel variant file (rsID, CHR, START, END in GRCh37)
- Tab-separated proxy variant file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2)
- VEP output .txt file for sentinels and proxies
- GTEx_Analysis_v7_eQTL.tar.gz cis-eQTL data from GTEx
- Expert-curated reference causal gene assignments (Supplementary Table S1: 227 mQTLs, 562 cis-pQTLs)

## Outputs

- OUTPUT_bottom_up_summary.txt
- OUTPUT_top_down_scores.txt
- OUTPUT_cooccurring_candidates.txt
- Sensitivity estimate (reference genes recovered, expected 223/227 ≈ 98%)
- Specificity estimate (true positives per prioritized candidate, expected 0.502)

## How to apply

Run the prioritization framework on the reference loci using the standard three input files (sentinel variants in GRCh37 rsID/CHR/START/END format, proxy variants with LEAD_rsID and r2, and VEP output), with default settings: ±500 kb local window, protein-coding gene biotype filter, proxy LD r2 threshold of 0.8, and all 44 GTEx tissues. Parse OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, and OUTPUT_cooccurring_candidates.txt, treating cooccurring (bottom-up AND top-down) genes as primary causal candidates. Compare the prioritized candidates against the 227 expert-curated reference causal genes, counting a locus as recovered when the curated gene appears in the prioritized set. Compute sensitivity as the fraction of reference genes recovered (target 223/227 ≈ 98%) and specificity as true positives per prioritized candidate (target 0.502), matching the published benchmark. Rationale: without a high-confidence reference causal gene set, tool sensitivity cannot be rigorously assessed, and cross-referencing mQTLs with pre-existing biochemical and genetic research provides such a set.

## Related tools

- **ProGeM** (R-script framework that prioritizes bottom-up, top-down, and cooccurring candidate causal genes at molecular QTLs, whose outputs are compared against the curated reference gene set) — https://github.com/ds763/ProGeM
- **R** (Language in which ProGeM is implemented and sourced via ProGeM_settings.R to execute the full pipeline)

## Examples

```
source("ProGeM_settings.R")
```

## Evaluation signals

- Sensitivity at the 227 mQTL loci equals 223/227 (~98%), i.e., the curated gene appears among ProGeM-prioritized candidates for all but 4 loci
- Specificity equals 0.502 true positives per prioritized candidate candidate gene set
- Cooccurring genes (bottom-up AND top-down) are treated as primary causal candidates in the comparison
- Pipeline processes ~213 sentinel variants in ~12-15 minutes on a standard desktop (4GB RAM, dual core i3), confirming a complete run
- All three output files (bottom_up_summary, top_down_scores, cooccurring_candidates) are populated for every sentinel rsID

## Limitations

- Reference causal gene assignments could be biased toward nearest genes, and 69% of causal candidates were nearest to the sentinel variant, so sensitivity may be inflated
- ProGeM and the curated mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed mQTL sensitivity may be inflated
- cis-eQTL data alone led to three false positives for every one true positive, so eQTL-only prioritization is a poor indicator of true causal genes

## Evidence

- [abstract] Applied to the 227 mQTL reference causal gene set, ProGeM identified the curated candidates for 223 of 227 mQTLs, demonstrating a sensitivity of 98%: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [intro] Reference set construction rationale: "it becomes possible to generate a large, high-confidence, mQTL-specific reference causal gene set, which can be used for the validation of both current and future causal gene prioritization tools"
- [methods] Benchmark parameters: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
- [readme] Cooccurring candidates as primary causal candidates: "These genes are so-called "concurrent" genes and should be considered primary causal candidates based on the fact that they satisfy both of the above-mentioned underlying assumptions of a causal gene."
- [discussion] Known circularity caveat: "the KEGG database was utilized both by ProGeM and as a guide for the expert literature review"
