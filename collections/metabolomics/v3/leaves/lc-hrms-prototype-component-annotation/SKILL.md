---
name: lc-hrms-prototype-component-annotation
description: Use when raw untargeted UPLC-HRMS peak lists of an herbal medicine (TCM)
  extract in positive and negative ESI modes must be reduced to a genuine prototype
  component list and annotated without prior structural prediction.
when_to_use_negative:
- The goal is in vivo metabolite identification and prototype-metabolite bridging
  rather than building the in vitro prototype list — that requires the MDRB post-processing
  technique instead of PATBS alone.
- Targeted analysis where component structures are already predicted/known and background
  subtraction is unnecessary.
- Data are already a curated, annotated feature table — re-running PATBS background
  subtraction would discard genuine features or add no value.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Environment in which the PATBS background-subtraction workflow and peak tabulation
    by ionization mode are executed; R is also the basis of the MDRB data post-processing
    platform
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Self-developed precise-and-thorough background-subtraction technique that
    strips background/blank ions from the fingerprint peak lists to retain genuine
    sample components non-targetedly
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing platform used downstream
    to bridge the in vitro prototype list to in vivo metabolites via m/z, mass defect,
    and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
- name: PubChem
  role: External compound database used to assign putative identities to annotated
    prototype components by m/z comparison
- name: ChemSpider
  role: External compound database used alongside PubChem for putative annotation
    of prototype components
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:5cb288a19f7644e87ff2f2aed0cbd9f7f8da6140ddffbf91765d5dee6da56a9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - ChemSpider
  - Chemspider
  - MDRB
  - PATBS
  - PubChem
  - Pubchem
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lc-hrms-prototype-component-annotation

## Summary

Annotates the prototype component inventory of an herbal extract from untargeted LC-HRMS fingerprint data in positive and negative ESI modes, using the PATBS background-subtraction technique in R and database comparison against PubChem and ChemSpider. It yields a mode-wise component count (e.g. 96 positive / 136 negative / 33 in both modes for CKXR) and an abundance-ranked prototype list.

## When to use

Use when raw untargeted UPLC-HRMS peak lists of an herbal medicine (TCM) extract in positive and negative ESI modes must be reduced to a genuine prototype component list and annotated without prior structural prediction.

## When NOT to use

- The goal is in vivo metabolite identification and prototype-metabolite bridging rather than building the in vitro prototype list — that requires the MDRB post-processing technique instead of PATBS alone.
- Targeted analysis where component structures are already predicted/known and background subtraction is unnecessary.
- Data are already a curated, annotated feature table — re-running PATBS background subtraction would discard genuine features or add no value.

## Inputs

- Raw UPLC-HRMS fingerprint peak list of the herbal extract in positive ESI mode
- Raw UPLC-HRMS fingerprint peak list of the herbal extract in negative ESI mode
- Blank/background QC runs for background subtraction by PATBS

## Outputs

- Prototype component table with putative identities annotated against PubChem and ChemSpider
- Mode-wise component counts (positive mode, negative mode, detected in both modes, total)
- Abundance-ranked prototype list (by extracted peak intensity)

## How to apply

Load the raw in vitro UPLC-HRMS fingerprint peak lists from both ESI modes into an R session and apply the precise-and-thorough background-subtraction (PATBS) technique to strip background/blank ions; PATBS is non-targeted and does not require prediction of component structures. Count the retained features per ionization mode and reconcile the positive-mode, negative-mode, and both-mode counts against the total (for CKXR: 96 + 136 with 33 in both = 199 prototypes). Annotate the retained m/z values by comparison against the PubChem and ChemSpider databases to assign putative identities to the prototype list. Rank annotated components by extracted peak intensity to identify the dominant prototypes (amygdalin and prunasin had the highest relative contents in CKXR). The resulting prototype table serves as the in vitro anchor that MDRB later bridges to in vivo metabolites via m/z, MS1/MS2 mass defect, and MS2 fragment similarity.

## Related tools

- **R** (Environment in which the PATBS background-subtraction workflow and peak tabulation by ionization mode are executed; R is also the basis of the MDRB data post-processing platform) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Self-developed precise-and-thorough background-subtraction technique that strips background/blank ions from the fingerprint peak lists to retain genuine sample components non-targetedly)
- **MDRB** (R-based mass dataset relevance bridging post-processing platform used downstream to bridge the in vitro prototype list to in vivo metabolites via m/z, mass defect, and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB
- **PubChem** (External compound database used to assign putative identities to annotated prototype components by m/z comparison)
- **ChemSpider** (External compound database used alongside PubChem for putative annotation of prototype components)

## Examples

```
Rscript patbs_fingerprint.R --pos pos_mode_peaks.csv --neg neg_mode_peaks.csv --db pubchem,chemspider --out prototype_list.csv
```

## Evaluation signals

- Rebuilt prototype table reproduces the reported mode-wise counts: total 199 components, 96 in positive ion mode, 136 in negative ion mode, 33 responding in both modes for CKXR
- Both-mode components (33) sum consistently with the positive/negative counts to the 199 total
- Amygdalin and prunasin appear at the top of the abundance ranking of annotated prototypes
- Every retained feature in the prototype list is traceable to a sample peak that survived PATBS background subtraction (no unexplained blank-origin ions)
- Putative identities are supported by matches in both PubChem and ChemSpider comparisons

## Limitations

- No specific mass spectrometry instrument model or data acquisition parameters are named in the source article, limiting reproducibility of acquisition settings
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy
- Putative annotations from PubChem/ChemSpider comparison are not validated against reference standards within this workflow
- The in vivo component identification built on this prototype list 'still' has 'more difficulties', per the article's own limitation statement

## Evidence

- [intro] Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS): "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [abstract] which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [intro] compared with Pubchem (https://pubchem.ncbi.nlm.nih.gov/) and Chemspider (https://www.chemspider.com/) databases: "compared with Pubchem (https://pubchem.ncbi.nlm.nih.gov/) and Chemspider (https://www.chemspider.com/) databases"
- [abstract] By combining MDRB with PATBS through this non-targeted identification technology: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy for identification, characterization and bridging analysis of TCM metabolite in"
- [discussion] No specific mass spectrometry instrument model or data acquisition parameters named: "the mass defect between MS1 and MS2, and the similarity of MS2 fragments"
