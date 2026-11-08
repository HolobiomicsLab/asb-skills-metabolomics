---
name: podp-validated-link-retrieval
description: 'Use when benchmarking a BGC–MS/MS linking method against known ground-truth
  links: paired omics datasets are registered in the PoDP, and you need to determine
  which validated links are detectable, retained after score filtering, and how their
  ranks change.'
when_to_use_negative:
- The datasets contain no PoDP-registered validated links, so no ground truth exists
  to benchmark against
- The analysis goal is de novo BGC–metabolite discovery rather than validation/benchmarking
  of an existing linking method
- Validated metabolites exceed the ~850 Da CANOPUS mass cut-off and lack class predictions,
  so NPClassScore filtering cannot be fairly assessed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3172
tools:
- name: NPLinker
  role: Platform for loading paired genomics/metabolomics data, computing standardised
    Metcalf scores, and applying NPClassScore filtering
  repo: github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical chemical class matching score used at a 0.25 cut-off to filter and
    re-rank candidate BGC–MS/MS links
  repo: github.com/louwenjjr/mibig_classifications
- name: GNPS
  role: Source of molecular networking data and mgf files for the MS/MS spectra of
    the PoDP datasets
- name: antiSMASH
  role: Genome mining for BGCs; antiSMASH 6 output merged with prior antiSMASH 3 data
    for 50 strains
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (GCFs) with a 0.3 cut-off, the candidate
    units scored by Metcalf
- name: CANOPUS
  role: Predicts structure-based compound classes from MS/MS spectra via Sirius v4.9.3,
    required for NPClassScore
  repo: github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:85f50574ecb4a7d89299d88d02bf9cb3de3aee6564970a226a5464b5198a06ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# podp-validated-link-retrieval

## Summary

Retrieves the 26 experimentally validated BGC–MS/MS genome-metabolome links recorded on the Paired Omics Data Platform (PoDP) for the Streptomyces/Salinispora (Crüsemann et al.), Cyanobacteria (MSV000084950) and Nocardia (MSV000084771) datasets, and benchmarks their survival through NPLinker's Metcalf co-occurrence and NPClassScore filtering.

## When to use

Use when benchmarking a BGC–MS/MS linking method against known ground-truth links: paired omics datasets are registered in the PoDP, and you need to determine which validated links are detectable, retained after score filtering, and how their ranks change.

## When NOT to use

- The datasets contain no PoDP-registered validated links, so no ground truth exists to benchmark against
- The analysis goal is de novo BGC–metabolite discovery rather than validation/benchmarking of an existing linking method
- Validated metabolites exceed the ~850 Da CANOPUS mass cut-off and lack class predictions, so NPClassScore filtering cannot be fairly assessed

## Inputs

- PoDP records of experimentally validated genome-metabolome links for the three datasets (26 links total)
- GNPS molecular networking output (mgf) for MSV000084950, MSV000084771 and the Crüsemann et al. Streptomyces/Salinispora dataset
- antiSMASH BGC annotations (merged antiSMASH 3 and antiSMASH 6 gbk outputs) for the dataset strains
- BiG-SCAPE GCF clustering (cut-off 0.3)
- CANOPUS/MolNetEnhancer structure-based class predictions for MS/MS spectra

## Outputs

- Per-link table of the 26 validated links with detectability status (24 detectable; retimycin and nocardimicin undetectable)
- Pass/fail record of Metcalf co-occurrence filtering (expect 5/26 failures, 75% full-workflow accuracy)
- Retention record after NPClassScore 0.25 filtering (expect 23/24 detectable links, 96%)
- Ranked positions of validated links before and after NPClassScore filtering, comparable to Table 1

## How to apply

Retrieve the 26 validated genome-metabolite links from the PoDP for the three datasets (Cyanobacteria MSV000084950, Nocardia MSV000084771, and the Streptomyces/Salinispora dataset from Crüsemann et al.). Match each validated compound to its MS/MS spectrum to establish detectability — retimycin and nocardimicin have no findable spectra, leaving 24 detectable links. Load the paired data into NPLinker, compute standardised Metcalf scores with a 2.5 cut-off (lowered to 2.0 for the Nocardia dataset, whose validated links scored low), and record which validated links pass co-occurrence filtering. Apply NPClassScore at its 0.25 default cut-off to surviving candidates and record which of the 24 detectable links are retained (expect 23, i.e. 96%). Record ranked positions of each validated link before and after NPClassScore filtering and compare against Table 1, including staurosporine and rosamicin moving to rank 1.

## Related tools

- **NPLinker** (Platform for loading paired genomics/metabolomics data, computing standardised Metcalf scores, and applying NPClassScore filtering) — github.com/NPLinker/nplinker
- **NPClassScore** (Empirical chemical class matching score used at a 0.25 cut-off to filter and re-rank candidate BGC–MS/MS links) — github.com/louwenjjr/mibig_classifications
- **GNPS** (Source of molecular networking data and mgf files for the MS/MS spectra of the PoDP datasets)
- **antiSMASH** (Genome mining for BGCs; antiSMASH 6 output merged with prior antiSMASH 3 data for 50 strains)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (GCFs) with a 0.3 cut-off, the candidate units scored by Metcalf)
- **CANOPUS** (Predicts structure-based compound classes from MS/MS spectra via Sirius v4.9.3, required for NPClassScore) — github.com/kaibioinfo/canopus_treemap

## Examples

```
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Retention fraction after NPClassScore 0.25 filtering equals 23/24 detectable validated links (96%)
- Full-workflow accuracy (Metcalf threshold) equals 21/26 passing, i.e. 5/26 links fail co-occurrence (75%)
- Exactly 2 of 26 validated compounds (retimycin, nocardimicin) cannot be matched to any MS/MS spectrum
- Per-link rank changes reproduce Table 1, with staurosporine and rosamicin at rank 1 after NPClassScore filtering
- Metcalf cut-off applied is 2.5 generally, lowered to 2.0 for the Nocardia dataset

## Limitations

- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the threshold to be lowered from 2.5 to 2, indicating weaker performance on the smallest dataset
- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions and their candidate links were discarded
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are not well handled
- No MS/MS spectrum could be found for retimycin and nocardimicin, so only 24 of 26 validated links are assessable
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce version inconsistency across 50 strains

## Evidence

- [abstract] 96% of experimentally validated links in three paired omics datasets were retained and prioritised by NPClassScore: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] Metcalf and NPClassScore cut-offs: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] Nocardia threshold adjustment: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold"
- [discussion] Undetectable validated compounds: "No MS/MS spectrum could be found this way for retimycin and nocardimicin."
- [discussion] Dataset accessions via PoDP: "The Cyanobacteria and Nocardia datasets were accessed through their PoDP accessions, MSV000084950 and MSV000084771"
- [discussion] Rank recording before/after filtering: "The ranked position for each validated link was recorded before and after filtering with NPClassScore"
