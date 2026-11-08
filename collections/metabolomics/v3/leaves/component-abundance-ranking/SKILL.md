---
name: component-abundance-ranking
description: Use when a non-targeted UPLC-HRMS fingerprint peak list (positive and
  negative ESI modes) must be tabulated by ionization mode and ranked by extracted
  peak intensity to nominate the most abundant prototype components, e.g. amygdalin
  and prunasin in CKXR.
when_to_use_negative:
- The goal is in vivo metabolite identification or metabolite–prototype bridging —
  that requires the MDRB scoring approach (m/z, MS1/MS2 mass defect, MS2 fragment
  similarity), not intensity ranking.
- Input is already a curated, ranked feature/abundance table with identities assigned
  — re-running the fingerprint tabulation adds nothing.
- Absolute quantification is required — this skill ranks uncalibrated extracted peak
  intensities from non-targeted fingerprint data, not validated quantitative assays.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: R session used to load the fingerprint dataset, tabulate features by ion mode,
    and rank components by extracted peak intensity
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Precise-and-thorough background-substraction strips background/blank ions
    from the fingerprint so only genuine sample components enter the abundance ranking
- name: Pubchem
  role: Database against which component m/z values are compared to assign putative
    prototype identities
- name: Chemspider
  role: Second database for m/z-based putative annotation of fingerprint components
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:9b0e8d6d1f99ac63b365599bcc73fa3c5a6f2eab722e30351a0ba59211e1d264
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# component-abundance-ranking

## Summary

Ranks components from a non-targeted UPLC-HRMS fingerprint (in vitro) by extracted peak intensity to nominate the most abundant prototype constituents — for CKXR this reproduced the 199-component inventory (96 positive / 136 negative / 33 dual-mode) with amygdalin and prunasin ranked highest.

## When to use

Use when a non-targeted UPLC-HRMS fingerprint peak list (positive and negative ESI modes) must be tabulated by ionization mode and ranked by extracted peak intensity to nominate the most abundant prototype components, e.g. amygdalin and prunasin in CKXR.

## When NOT to use

- The goal is in vivo metabolite identification or metabolite–prototype bridging — that requires the MDRB scoring approach (m/z, MS1/MS2 mass defect, MS2 fragment similarity), not intensity ranking.
- Input is already a curated, ranked feature/abundance table with identities assigned — re-running the fingerprint tabulation adds nothing.
- Absolute quantification is required — this skill ranks uncalibrated extracted peak intensities from non-targeted fingerprint data, not validated quantitative assays.

## Inputs

- PATBS-cleaned UPLC-HRMS fingerprint peak list in positive ESI mode (m/z + extracted peak intensity)
- PATBS-cleaned UPLC-HRMS fingerprint peak list in negative ESI mode (m/z + extracted peak intensity)
- PubChem and ChemSpider database entries for m/z-based putative annotation

## Outputs

- Mode-wise component inventory table (counts per ion mode, dual-mode assignments; 199 total for CKXR)
- Prototype list annotated with putative identities from PubChem/ChemSpider
- Abundance ranking of prototype components by extracted peak intensity (top: amygdalin, prunasin)

## How to apply

Load the in vitro UPLC-HRMS fingerprint peak list (positive and negative ESI modes) into R, after PATBS background subtraction has stripped blank/background ions so only genuine sample features remain — PATBS was chosen because non-targeted techniques 'do not require prediction of the structure of herbal components'. Tabulate retained features by ionization mode and reconcile counts against the expected split (96 positive, 136 negative, 33 in both modes, summing to 199 total). Annotate component m/z values against PubChem (https://pubchem.ncbi.nlm.nih.gov/) and ChemSpider (https://www.chemspider.com/) to assign putative prototype identities. Rank annotated components by extracted peak intensity to produce the abundance table; the decision point is that the top-ranked entries (here amygdalin and prunasin) define the prototype list that downstream MDRB bridging will link to in vivo metabolites, since 'the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB'.

## Related tools

- **R** (R session used to load the fingerprint dataset, tabulate features by ion mode, and rank components by extracted peak intensity) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Precise-and-thorough background-substraction strips background/blank ions from the fingerprint so only genuine sample components enter the abundance ranking)
- **Pubchem** (Database against which component m/z values are compared to assign putative prototype identities)
- **Chemspider** (Second database for m/z-based putative annotation of fingerprint components)

## Examples

```
peaks <- read.csv("ckxr_fingerprint_peaks.csv"); peaks[order(-peaks$intensity), c("component","mode","intensity")]
```

## Evaluation signals

- Mode-wise counts reconcile exactly: 96 positive-mode + 136 negative-mode with 33 detected in both modes, summing to 199 total components.
- The top of the abundance ranking lists amygdalin and prunasin as the highest relative contents.
- Every retained feature is traceable to a sample (non-blank) signal, i.e. PATBS background subtraction removed blank ions before counting.
- Top-ranked components carry putative annotations supported by m/z matches in both PubChem and ChemSpider.
- The prototype table produced is usable as the prototype database for downstream MDRB metabolite-prototype bridging.

## Limitations

- Non-targeted fingerprinting does not require prior structural prediction, so assignments are putative until confirmed by MS/MS or standards.
- The MDRB platform provides no changelog or stated software versions, so reproducibility of the data-processing environment must be pinned manually.
- Abundance ranking reflects extracted peak intensities in the fingerprint only; relative contents may not translate directly to in vivo exposure.

## Evidence

- [intro] Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components.: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [intro] compared with Pubchem (https://pubchem.ncbi.nlm.nih.gov/) and Chemspider (https://www.chemspider.com/) databases: "compared with Pubchem (https://pubchem.ncbi.nlm.nih.gov/) and Chemspider (https://www.chemspider.com/) databases"
- [abstract] By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy"
- [intro] the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB: "the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB"
