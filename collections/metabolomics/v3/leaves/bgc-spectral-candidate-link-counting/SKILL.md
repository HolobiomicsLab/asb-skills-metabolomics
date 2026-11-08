---
name: bgc-spectral-candidate-link-counting
description: Use when quantifying how many candidate BGC–MS/MS spectrum links each
  GCF retains under co-occurrence-only scoring versus chemical-class filtering, e.g.
  to reproduce or assess manageable candidate-link histograms.
when_to_use_negative:
- When the goal is to prioritise or validate individual links rather than summarise
  per-GCF candidate-link burden (the ranked-position validation workflow is the appropriate
  skill instead)
- When MS/MS spectra lack CANOPUS or MolNetEnhancer structure-based class predictions
  and you intend to keep those links — NPClassScore filtering discards candidate links
  without structure-based classes
- When dataset-specific score calibration is needed, e.g. the Nocardia dataset where
  the standardised Metcalf threshold had to be lowered from 2.5 to 2 to retain validated
  links
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform that integrates GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs,
    and scoring; provides Metcalf scoring, NPClassScore filtering, and get_links for
    extracting candidate BGC–MS/MS links
  repo: github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical class-matching score (cut-off 0.25) used to filter candidate links
    after Metcalf scoring; filtering table built from MIBiG 2.0 classifications
  repo: github.com/louwenjjr/mibig_classifications
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into gene cluster families (GCFs) at a cut-off of 0.3,
    the units whose per-family link counts are tallied
- name: GNPS
  role: Source of molecular networking MS/MS data integrated by NPLinker; class predictions
    retrieved through the GNPS API
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:b2dd70e8a32205c4fd827c4a8b4b5fd44f2dca12f3e4477f9236f2abb3af5929
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPClassScore
  - NPLinker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bgc-spectral-candidate-link-counting

## Summary

Counts the number of candidate BGC–MS/MS spectrum links per gene cluster family (GCF) in NPLinker, before and after NPClassScore chemical-class filtering, to quantify how class matching prunes candidate lists (e.g. from 42 to 270 GCFs with ≤25 links in the Streptomyces/Salinispora dataset).

## When to use

Use when quantifying how many candidate BGC–MS/MS spectrum links each GCF retains under co-occurrence-only scoring versus chemical-class filtering, e.g. to reproduce or assess manageable candidate-link histograms.

## When NOT to use

- When the goal is to prioritise or validate individual links rather than summarise per-GCF candidate-link burden (the ranked-position validation workflow is the appropriate skill instead)
- When MS/MS spectra lack CANOPUS or MolNetEnhancer structure-based class predictions and you intend to keep those links — NPClassScore filtering discards candidate links without structure-based classes
- When dataset-specific score calibration is needed, e.g. the Nocardia dataset where the standardised Metcalf threshold had to be lowered from 2.5 to 2 to retain validated links

## Inputs

- NPLinker local-mode example dataset from Zenodo record 10822604 (nplinker_local_mode_example.zip)
- GNPS molecular networking data (molecular network, mgf, mzXML files)
- antiSMASH BGC annotations grouped into GCFs (BiG-SCAPE output, cut-off 0.3)
- Structure-based chemical class predictions for spectra (CANOPUS via SIRIUS on the GNPS mgf, or MolNetEnhancer run on the GNPS platform)
- MIBiG-derived chemical classes for BGCs

## Outputs

- Per-GCF candidate link counts before filtering (pre-filtering histogram)
- Per-GCF candidate link counts after NPClassScore >= 0.25 filtering (post-filtering histogram)
- Side-by-side matplotlib histogram of link counts per GCF, highlighting GCFs with <=10 and <=25 candidate links

## How to apply

Load the example dataset (Zenodo record 10822604) into NPLinker with GNPS molecular networking data, antiSMASH BGCs, and strain mappings, then run standardised Metcalf co-occurrence scoring with a cut-off of 2.5 to obtain the unfiltered candidate link set. Count links per GCF to build the pre-filtering histogram. Apply NPClassScore, which uses structure-based chemical classes (CANOPUS/MolNetEnhancer predictions for spectra, MIBiG-derived classes for BGCs), and discard candidate links without structure-based classes (the filter_missing_scores toggle). Retain only links with NPClassScore >= 0.25 and re-count links per GCF to build the post-filtering histogram. Plot both per-GCF link-count histograms side by side (e.g. with matplotlib) to visualise the reduction; in the Streptomyces/Salinispora dataset the class-based filtering reduced links by on average 63%.

## Related tools

- **NPLinker** (Platform that integrates GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs, and scoring; provides Metcalf scoring, NPClassScore filtering, and get_links for extracting candidate BGC–MS/MS links) — github.com/NPLinker/nplinker
- **NPClassScore** (Empirical class-matching score (cut-off 0.25) used to filter candidate links after Metcalf scoring; filtering table built from MIBiG 2.0 classifications) — github.com/louwenjjr/mibig_classifications
- **BiG-SCAPE** (Groups antiSMASH BGCs into gene cluster families (GCFs) at a cut-off of 0.3, the units whose per-family link counts are tallied)
- **GNPS** (Source of molecular networking MS/MS data integrated by NPLinker; class predictions retrieved through the GNPS API)

## Examples

```
from nplinker import load_config; npl = load_config('nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Post-filtering histogram shows 92 GCFs with <=10 candidate links and 270 GCFs with <=25 candidate links in the Streptomyces/Salinispora dataset
- Pre-filtering (co-occurrence-only, Metcalf cut-off 2.5) histogram shows 5 GCFs with <=10 links and 42 GCFs with <=25 links
- Link reduction after NPClassScore filtering matches the reported on-average 63% reduction versus co-occurrence-only strategy
- Candidate links lacking structure-based classes (from the 1988 spectra without CANOPUS/MolNetEnhancer predictions) are excluded when the filter_missing_scores toggle is set
- Both histograms are computed over the same GCF set so the before/after comparison is like-for-like

## Limitations

- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions; candidate links to these spectra are discarded by class filtering, and not filtering them results in an average decrease of 49% of candidate links per GCF
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are not well handled
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, e.g. on the GNPS platform
- Dataset-specific thresholds may be needed: for the Nocardia dataset the standardised Metcalf threshold was changed from 2.5 to 2 to include validated links

## Evidence

- [discussion] Standardised Metcalf scoring with NPClassScore cut-off: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] Discarding links without structure-based classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [abstract] Average link reduction by NPClassScore: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [methods] Zenodo example dataset download: "url="https://zenodo.org/records/10822604/files/nplinker_local_mode_example.zip""
- [discussion] MolNetEnhancer must run externally: "Currently, MolNetEnhancer has to be run externally, for example on the GNPS platform"
- [readme] NPClassScore purpose in NPLinker: [evidence span withheld — anchor status: external_doc]
