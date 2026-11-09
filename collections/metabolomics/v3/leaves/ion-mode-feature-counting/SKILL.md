---
name: ion-mode-feature-counting
description: Use when you have UPLC-HRMS peak lists acquired in both positive and
  negative ESI modes and need a non-targeted count of genuine sample components per
  mode, reconciled against background-subtracted totals.
when_to_use_negative:
- The peak table has already been background-subtracted and filtered, so PATBS would
  double-process the data and distort mode-wise counts
- Only one ion mode was acquired, so the positive/negative/both-mode reconciliation
  (96/136/33) cannot be performed
- The goal is targeted quantification of pre-selected analytes rather than non-targeted
  fingerprint component enumeration
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PATBS
  role: Background-subtraction step that strips blank/background ions from the ESI-mode
    peak lists before counting retained components, without requiring prior structure
    prediction
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing that yields the pooled
    plasma peak-area table from which per-mode relative abundance shares are computed
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Environment for loading the peak tables, tabulating features by ion mode,
    computing percentage shares, and writing the summary CSV
- name: Pubchem
  role: Database queried to assign putative identities to retained component m/z values
    in the prototype list
- name: Chemspider
  role: Second database compared against for m/z annotation of prototype components
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:10a6e0ee477980b89b4d398dbec70c454f45929d6f1f9adfaace2da925074f3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - Chemspider
  - MDRB
  - PATBS
  - Pubchem
  - R
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ion-mode-feature-counting

## Summary

Counts and reconciles UPLC-HRMS chemical components per electrospray ionization mode after PATBS background subtraction, producing a mode-wise prototype inventory (e.g. CKXR: 199 total; 96 positive / 136 negative / 33 in both modes) with database-annotated identities and abundance ranking.

## When to use

Use when you have UPLC-HRMS peak lists acquired in both positive and negative ESI modes and need a non-targeted count of genuine sample components per mode, reconciled against background-subtracted totals.

## When NOT to use

- The peak table has already been background-subtracted and filtered, so PATBS would double-process the data and distort mode-wise counts
- Only one ion mode was acquired, so the positive/negative/both-mode reconciliation (96/136/33) cannot be performed
- The goal is targeted quantification of pre-selected analytes rather than non-targeted fingerprint component enumeration

## Inputs

- Raw UPLC-HRMS fingerprint peak lists in positive and negative ESI modes (in vitro sample plus blank/background runs)
- MDRB post-processed pooled plasma peak-area table with feature-level peak areas per ion mode (feature IDs such as H88/H97 positive, M197/M215 negative)
- Reference structure databases: PubChem and ChemSpider for m/z annotation

## Outputs

- Mode-wise prototype component inventory (total count with positive/negative/both-mode split, e.g. 199 = 96/136/33)
- Annotated prototype list with putative identities from PubChem/ChemSpider and abundance ranking
- cyanogenic_glycoside_relative_abundance.csv with columns for ion mode, feature IDs, summed peak area, and relative abundance percentage
- Summary flagging discrepancies between recomputed and reported counts/shares

## How to apply

Load the raw in vitro UPLC-HRMS fingerprint peak lists for both ESI modes into R and apply the precise-and-thorough background-subtraction (PATBS) technique to strip blank/background ions without requiring prior structural prediction. Tabulate the retained features by ionization mode, counting components detected in positive mode, negative mode, and both modes, and check the reconciliation invariant (here 96 + 136 − 33 = 199). Annotate retained m/z values against PubChem and ChemSpider to assign putative prototype identities. For exposure work, take the MDRB post-processed pooled plasma peak-area table, sum the cyanogenic glycoside feature rows (amygdalin H88/M197, prunasin H97/M215) and divide by total peak area per mode to obtain relative abundance shares (43.16% positive, 53.57% negative). Rank annotated prototypes by extracted peak intensity to confirm the highest-content components (amygdalin and prunasin for CKXR); flag any discrepancy from expected counts or shares in the output summary.

## Related tools

- **PATBS** (Background-subtraction step that strips blank/background ions from the ESI-mode peak lists before counting retained components, without requiring prior structure prediction)
- **MDRB** (R-based mass dataset relevance bridging post-processing that yields the pooled plasma peak-area table from which per-mode relative abundance shares are computed) — https://github.com/933ZhangDD/MDRB
- **R** (Environment for loading the peak tables, tabulating features by ion mode, computing percentage shares, and writing the summary CSV)
- **Pubchem** (Database queried to assign putative identities to retained component m/z values in the prototype list)
- **Chemspider** (Second database compared against for m/z annotation of prototype components)

## Examples

```
Rscript -e 'd<-read.csv("pooled_plasma_peak_area.csv"); agg<-aggregate(peak_area~ion_mode,data=d[d$feature_id%in%c("H88","H97","M197","M215"),],sum); agg$pct<-100*agg$peak_area/sum(d$peak_area)'
```

## Evaluation signals

- Mode-wise counts reconcile arithmetically: positive (96) + negative (136) − both (33) = 199 total components
- Abundance ranking places amygdalin and prunasin at the top of the prototype list by extracted peak intensity
- Recomputed cyanogenic glycoside share matches 43.16% (positive mode) and 53.57% (negative mode) within ±0.5 percentage points
- The relative-abundance CSV contains the required columns (ion mode, feature IDs, summed peak area, percentage) and populates H88/H97/M197/M215 feature IDs
- Any feature-count or share deviation from reported values is explicitly flagged in the output summary rather than silently accepted

## Limitations

- No software versions are stated for the MDRB technique or the data-processing platform, so reproducibility of exact feature counts may vary
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the counting/annotation strategy
- No specific mass spectrometry instrument model or data acquisition parameters are named, limiting transferability of count thresholds across instruments

## Evidence

- [intro] Non-targeted techniques such as PATBS need no prior structure prediction: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [intro] MDRB is an R-based post-processing technique integrating m/z, MS2 matching, and mass defect: "we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB) data post-processing"
- [abstract] MDRB builds a TCM component relationship network and improves identification efficiency: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency"
- [abstract] The strategy was demonstrated on CKXR exposure, metabolism, and disposition in mice: "we adopted the proposed strategy to investigate the process of exposure, metabolism, and disposition of Semen Armeniacae Amarum (CKXR) in mice"
