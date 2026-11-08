---
name: link-reduction-percentage-calculation
description: Use when you need to quantify the pruning effect of NPClassScore class
  matching on candidate BGC-MS/MS links per GCF in paired genome-metabolome datasets
  after Metcalf scoring, and to validate reported reduction percentages.
when_to_use_negative:
- 'The goal is absolute prioritisation performance rather than reduction accounting:
  without discarding links that lack structure-based classes, the average decrease
  is only 49%, so mixing filter_missing_scores settings makes the percentage incomparable
  to the reported 63%.'
- 'The Nocardia dataset is analysed without adjusting the Metcalf threshold: validated
  links there scored below the standard cut-off of 2.5, so the threshold must be lowered
  to 2 to include them before counting reductions.'
- 'No structure-based class predictions exist (no CANOPUS/MolNetEnhancer output):
  NPClassScore filtering will discard nearly all links and the reduction percentages
  are not meaningful.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform that loads PoDP paired data, computes standardised Metcalf links,
    applies NPClassScore filtering, and exposes get_links for candidate link enumeration
  repo: github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class-matching score used to filter Metcalf candidate links below
    a cut-off of 0.25
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Source of molecular networking MS/MS data and of ClassyFire/NPClassifier classifications
    retrieved through the GNPS API
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (GCFs) that define the per-GCF link
    counting unit
- name: antiSMASH
  role: Provides genome-mined BGCs (antiSMASH 6 merged with antiSMASH 3 data from
    the other 50 strains)
- name: CANOPUS
  role: Predicts structure-based classes from MS/MS spectra via Sirius on the GNPS
    mgf, needed to avoid discarding links
  repo: github.com/kaibioinfo/canopus_treemap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:550069ac662a4f7d70358d78408826da63d20655aab9826d61426f2e68ceca3f
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# link-reduction-percentage-calculation

## Summary

Quantifies how much NPClassScore chemical-class filtering (cut-off 0.25) reduces candidate BGC–MS/MS spectrum links per GCF relative to standardised Metcalf co-occurrence scoring alone, across the three PoDP paired omics datasets.

## When to use

Use when you need to quantify the pruning effect of NPClassScore class matching on candidate BGC-MS/MS links per GCF in paired genome-metabolome datasets after Metcalf scoring, and to validate reported reduction percentages.

## When NOT to use

- The goal is absolute prioritisation performance rather than reduction accounting: without discarding links that lack structure-based classes, the average decrease is only 49%, so mixing filter_missing_scores settings makes the percentage incomparable to the reported 63%.
- The Nocardia dataset is analysed without adjusting the Metcalf threshold: validated links there scored below the standard cut-off of 2.5, so the threshold must be lowered to 2 to include them before counting reductions.
- No structure-based class predictions exist (no CANOPUS/MolNetEnhancer output): NPClassScore filtering will discard nearly all links and the reduction percentages are not meaningful.

## Inputs

- nplinker.toml configuration per PoDP dataset (podp mode with the dataset's PoDP id, e.g. MSV000084950, MSV000084771, and the Streptomyces/Salinispora accessions MSV000078836, MSV000078839, MSV000079284)
- GNPS molecular networking output (mgf and network files) downloaded and extracted via NPLinker's GNPSDownloader/GNPSExtractor
- antiSMASH 3/6 merged BGC annotations and BiG-SCAPE GCFs (BiG-SCAPE cut-off 0.3)
- Strain mappings from PoDP and NCBI RefSeq/GenBank genome identifiers
- Structure-based classes from CANOPUS (Sirius v4.9.3 on the GNPS mgf) and MolNetEnhancer
- List of experimentally validated BGC-metabolite links for ranked-position recording

## Outputs

- Candidate link counts per GCF before Metcalf/NPClassScore filtering and after NPClassScore filtering (e.g. Cyanobacteria 27 to 13; Nocardia 206 to 64)
- Per-dataset percentage reductions (68% Streptomyces/Salinispora, 53% Cyanobacteria, 69% Nocardia) and the 63% average decrease
- Ranked positions of validated links before and after NPClassScore filtering
- Link-count tables written to output

## How to apply

For each PoDP dataset (154 Streptomyces/Salinispora, 24 cyanobacterial, and 11 Nocardia strains), instantiate NPLinker with a nplinker.toml in podp mode and call load_data() to assemble GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs, and strain mappings. Compute candidate links with npl.get_links(npl.gcfs, 'metcalf') using a standardised Metcalf cut-off of 2.5 (lowered to 2 for the Nocardia dataset because validated links had low Metcalf scores). Filter those links with NPClassScore at a cut-off of 0.25, discarding links lacking structure-based classes (1988 MS/MS spectra across the datasets had no CANOPUS or MolNetEnhancer prediction). Count candidate links per GCF before and after filtering, compute per-dataset percentage reductions and their average, and compare against the reported 68% (Streptomyces/Salinispora), 53% (Cyanobacteria), 69% (Nocardia), and 63% average. Also record the ranked positions of experimentally validated links before and after filtering to confirm prioritisation.

## Related tools

- **NPLinker** (Platform that loads PoDP paired data, computes standardised Metcalf links, applies NPClassScore filtering, and exposes get_links for candidate link enumeration) — github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class-matching score used to filter Metcalf candidate links below a cut-off of 0.25) — github.com/NPLinker/nplinker
- **GNPS** (Source of molecular networking MS/MS data and of ClassyFire/NPClassifier classifications retrieved through the GNPS API)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (GCFs) that define the per-GCF link counting unit)
- **antiSMASH** (Provides genome-mined BGCs (antiSMASH 6 merged with antiSMASH 3 data from the other 50 strains))
- **CANOPUS** (Predicts structure-based classes from MS/MS spectra via Sirius on the GNPS mgf, needed to avoid discarding links) — github.com/kaibioinfo/canopus_treemap

## Examples

```
from nplinker.nplinker import load_config; npl = load_config('nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Reproduced per-dataset reductions match the reported 68% (Streptomyces/Salinispora), 53% (Cyanobacteria, 27 to 13 links per GCF), and 69% (Nocardia, 206 to 64 links per GCF).
- Average decrease in candidate links per GCF across the three datasets equals the reported 63%.
- 96% of experimentally validated links are retained and prioritised (ranked positions improve or are preserved) after NPClassScore filtering.
- The Metcalf cut-off settings match the article: 2.5 for the two larger datasets, 2 for the Nocardia dataset.
- The share of discarded links lacking CANOPUS or MolNetEnhancer predictions is consistent (1988 MS/MS spectra across datasets without a prediction).

## Limitations

- Without discarding links missing structure-based classes, the average candidate-link reduction drops to 49%, so the 63% figure depends on the filter_missing_scores toggle.
- No MS/MS spectrum could be found for the validated compounds retimycin and nocardimicin, so they cannot contribute to the validation counts.
- The Nocardia dataset required lowering the standardised Metcalf threshold from 2.5 to 2, indicating the method performs less well on the smallest dataset (11 strains).
- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled by current tools, and MolNetEnhancer must be run externally on the GNPS platform rather than within NPLinker.

## Evidence

- [abstract] the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] Standardised Metcalf scoring cut-off and NPClassScore filtering thresholds: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] Nocardia Metcalf threshold adjustment: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold"
- [discussion] Unfiltered-links alternative reduction: "Not filtering out such links results in an average decrease of 49% of the candidate links per GCF"
- [methods] NPLinker invocation for link computation: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
