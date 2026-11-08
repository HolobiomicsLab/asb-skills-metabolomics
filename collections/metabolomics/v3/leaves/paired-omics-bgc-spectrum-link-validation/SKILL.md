---
name: paired-omics-bgc-spectrum-link-validation
description: Use when a paired omics (genome + metabolomics) dataset contains curated
  validated genome-metabolome links and you must quantify how many survive Metcalf
  and NPClassScore filtering, or verify prioritisation/rank improvements of known
  links.
when_to_use_negative:
- Datasets with no curated/validated genome-metabolome links — the skill measures
  retention of known links, not discovery of new ones
- Metabolites >850 Da or spectra lacking CANOPUS/MolNetEnhancer predictions, where
  NPClassScore classes are unavailable and links are discarded by design
- Analyses needing structure-similarity scoring (e.g. MS2DeepScore/Spec2Vec) rather
  than chemical-class matching
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform that loads paired genomics/metabolomics data, computes standardised
    Metcalf scores, and applies NPClassScore filtering and link ranking
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical chemical class matching score (cut-off 0.25) used to filter candidate
    BGC–MS/MS links after Metcalf scoring
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of molecular networking output (mgf files) consumed by NPLinker; ClassyFire/NPClassifier
    classifications retrieved via GNPS API
- name: antiSMASH
  role: Generates BGC predictions (v6 merged with prior v3 data) used as the genomic
    side of candidate links
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (cut-off 0.3) that are the units scored
    against spectra
- name: CANOPUS
  role: SIRIUS-based structure-class prediction run directly on the GNPS mgf file
    to supply spectrum-side classes
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: mibig_classifications
  role: Provides MIBiG-based class-matching scoring tables underlying NPClassScore
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:9690690212eb14f892fc2b90432eca4bed4889867725c25514591fa1da28d60f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - NPClassScore
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# paired-omics-bgc-spectrum-link-validation

## Summary

Reproduce and validate experimentally verified BGC–MS/MS spectrum links from PoDP datasets by running NPLinker's standardised Metcalf co-occurrence scoring followed by NPClassScore chemical-class filtering, recording retention fractions and rank changes of validated links.

## When to use

Use when a paired omics (genome + metabolomics) dataset contains curated validated genome-metabolome links and you must quantify how many survive Metcalf and NPClassScore filtering, or verify prioritisation/rank improvements of known links.

## When NOT to use

- Datasets with no curated/validated genome-metabolome links — the skill measures retention of known links, not discovery of new ones
- Metabolites >850 Da or spectra lacking CANOPUS/MolNetEnhancer predictions, where NPClassScore classes are unavailable and links are discarded by design
- Analyses needing structure-similarity scoring (e.g. MS2DeepScore/Spec2Vec) rather than chemical-class matching

## Inputs

- PoDP dataset records with validated BGC–metabolite links (e.g. MSV000084950, MSV000084771, Crüsemann Streptomyces/Salinispora dataset)
- GNPS molecular networking output (mgf file and network files) for each MS/MS dataset
- antiSMASH BGC/GCF predictions (antiSMASH 6 merged with prior antiSMASH 3 data; BiG-SCAPE GCF clustering at 0.3 cut-off)
- CANOPUS (via SIRIUS, e.g. v4.9.3) and MolNetEnhancer structure-based class predictions for spectra
- MIBiG-derived NPClassScore scoring tables (github.com/louwenjjr/mibig_classifications)

## Outputs

- Per-validated-link pass/fail table for Metcalf (2.5 or 2.0) and NPClassScore (0.25) thresholds
- Retention fraction of validated links after NPClassScore filtering (e.g. 23/24 = 96%)
- Full-workflow accuracy (e.g. 21/26 passing both filters = 75% given 5/26 below Metcalf cut-off)
- Ranked positions of validated links before and after NPClassScore filtering (Table 1-style rank comparison)

## How to apply

Retrieve the validated genome-metabolome links from the Paired Omics Data Platform for the target datasets (e.g. Cyanobacteria MSV000084950, Nocardia MSV000084771, Streptomyces/Salinispora from Crüsemann et al.) and load paired data into NPLinker via load_config/load_data. Compute standardised Metcalf scores for candidate GCF-to-spectrum links with a cut-off of 2.5, lowering to 2.0 for datasets (like Nocardia) where validated links score low. Filter surviving candidate links with NPClassScore at its default 0.25 cut-off, discarding links lacking CANOPUS/MolNetEnhancer structure-based classes (filter_missing_scores toggle). For each validated link, record whether it passed Metcalf, whether it was retained by NPClassScore, and its ranked position before and after filtering; compare against expected values (e.g. 23/24 detectable links retained = 96%; 5/26 failing Metcalf = 75% full-workflow accuracy).

## Related tools

- **NPLinker** (Platform that loads paired genomics/metabolomics data, computes standardised Metcalf scores, and applies NPClassScore filtering and link ranking) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Empirical chemical class matching score (cut-off 0.25) used to filter candidate BGC–MS/MS links after Metcalf scoring) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of molecular networking output (mgf files) consumed by NPLinker; ClassyFire/NPClassifier classifications retrieved via GNPS API)
- **antiSMASH** (Generates BGC predictions (v6 merged with prior v3 data) used as the genomic side of candidate links)
- **BiG-SCAPE** (Groups BGCs into gene cluster families (cut-off 0.3) that are the units scored against spectra)
- **CANOPUS** (SIRIUS-based structure-class prediction run directly on the GNPS mgf file to supply spectrum-side classes) — https://github.com/kaibioinfo/canopus_treemap
- **mibig_classifications** (Provides MIBiG-based class-matching scoring tables underlying NPClassScore) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Retention fraction after NPClassScore filtering matches 96% (23 of 24 detectable validated links)
- Full-workflow accuracy matches 75% (5 of 26 validated links fall below the Metcalf threshold)
- Exactly 2 of 26 validated compounds (retimycin, nocardimicin) have no detectable MS/MS spectrum
- Per-link rank changes reproduce Table 1, including staurosporine and rosamicin moving to rank 1 after filtering
- Nocardia dataset validated links require the lowered Metcalf threshold of 2.0 rather than the default 2.5

## Limitations

- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions and their links were discarded; without this filtering the average candidate links per GCF decreases by only 49%
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are poorly handled
- The Nocardia dataset exhibited quite low standardised Metcalf scores for validated links, requiring the threshold to be lowered from 2.5 to 2
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce a version inconsistency across 50 strains
- MolNetEnhancer must be run externally on the GNPS platform, not within NPLinker

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] 96% of validated links retained: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] Nocardia Metcalf threshold lowered: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold"
- [discussion] Missing spectra for retimycin and nocardimicin: "No MS/MS spectrum could be found this way for retimycin and nocardimicin."
- [discussion] Ranked positions recorded before and after filtering: "The ranked position for each validated link was recorded before and after filtering with NPClassScore"
- [methods] NPLinker API usage from README/methods: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
