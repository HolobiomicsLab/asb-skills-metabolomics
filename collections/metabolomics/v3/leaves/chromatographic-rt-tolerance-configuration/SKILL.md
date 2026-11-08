---
name: chromatographic-rt-tolerance-configuration
description: Use when constructing metScribeR reference libraries from mzML files
  of authentic standards and you must choose the m/z and RT tolerances for EIC extraction
  and adduct distinction.
when_to_use_negative:
- 'The task requires MS/MS-based identification: metScribeR focuses library building
  on MS1 and RT data and does not use or require MS2 spectra for library construction'
- 'You need untargeted feature detection or RT prediction across instruments: RT is
  not consistently transferable across different LC-MS conditions, so these tolerances
  only apply to within-laboratory, method-specific data'
- The data are already processed feature tables or peak lists rather than raw mzML
  files of authentic standards
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: metScribeR
  role: R/Shiny package whose user_data_input() step accepts the eic_mz_tolerance
    and eic_rt_tolerance values that drive EIC creation and adduct distinction
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: Chromatogram-handling dependency installed alongside metScribeR (devtools::install_github('ethanbass/chromatographR'))
  repo: ethanbass/chromatographR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:8456ff178f3b83f51e311115d6cecaa177b35588170f698428fb61e11177f1de
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - R
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatographic-rt-tolerance-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Setting the eic_mz_tolerance and eic_rt_tolerance parameters in metScribeR's user_data_input() so EICs are extracted at expected adduct m/z values and adducts of the same standard are distinguished. Tolerances must be grounded in the LC-MS instrument's ability to confidently separate two signals.

## When to use

Use when constructing metScribeR reference libraries from mzML files of authentic standards and you must choose the m/z and RT tolerances for EIC extraction and adduct distinction.

## When NOT to use

- The task requires MS/MS-based identification: metScribeR focuses library building on MS1 and RT data and does not use or require MS2 spectra for library construction
- You need untargeted feature detection or RT prediction across instruments: RT is not consistently transferable across different LC-MS conditions, so these tolerances only apply to within-laboratory, method-specific data
- The data are already processed feature tables or peak lists rather than raw mzML files of authentic standards

## Inputs

- standards_df.csv (or .tsv) with columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path
- adduct_df.csv (or .tsv) with columns adduct, change_from_neutral, mode (POS/NEG)
- Positive- and negative-mode mzML files (converted with MSConvert) for each authentic standard
- User-supplied eic_mz_tolerance and eic_rt_tolerance values
- mzML_parsing_noise_level read from the noise plot

## Outputs

- Per-adduct EIC chromatograms stored in the metScribeR storage object for every standard-adduct pair within the m/z tolerance
- storage_object_initial.RDS capturing the post-EIC state so downstream peak-finding can be restarted without repeating the initial computation

## How to apply

At the metScribeR input step, choose an m/z tolerance and an RT tolerance for creating EICs and distinguishing between adducts; the README states these tolerances should be set based on the ability for the LC-MS equipment to confidently separate two signals, so inspect instrument-level peak separation (e.g. on the noise plot and 'Find Peaks' figures) rather than using arbitrary defaults. The m/z tolerance governs filtering of raw MS1 observations around each expected adduct m/z (neutral monoisotopic_mass plus adduct change_from_neutral, grouped by POS/NEG mode) before data are grouped by time and summed into per-adduct chromatograms. The RT tolerance is reused downstream: additional adducts from a standard must elute within this user-defined tolerance of the assigned RT for the standard, which underlies the check-for-crossed-adducts identification probability logic. Related parameters set in the same dialog are the mzML_parsing_noise_level (read off the noise plot; all MS observations below it are dropped) and the output directory. Because RT is only reproducible within a single laboratory's LC-MS method, re-derive tolerances whenever the chromatographic method changes rather than porting them between setups.

## Related tools

- **metScribeR** (R/Shiny package whose user_data_input() step accepts the eic_mz_tolerance and eic_rt_tolerance values that drive EIC creation and adduct distinction) — https://github.com/ncats/metScribeR
- **chromatographR** (Chromatogram-handling dependency installed alongside metScribeR (devtools::install_github('ethanbass/chromatographR'))) — ethanbass/chromatographR

## Examples

```
library(metScribeR); setwd(system.file('extdata', package = 'metScribeR')); runMetScribeRShinyApp()
```

## Evaluation signals

- storage_object_initial.RDS exists on disk and reloads as a metScribeR storage object containing EIC chromatograms for every standards_df row with at least one non-missing mzML file path
- Each EIC chromatogram corresponds to an expected adduct m/z computed as monoisotopic_mass plus change_from_neutral within the configured m/z tolerance, grouped by POS/NEG mode
- Tolerances are defensible in that the LC-MS equipment can confidently separate two signals at the chosen m/z and RT offsets (inspect the 'Find Peaks' figures and descriptive boxplots)
- In cross-checking, additional adducts of a standard elute within the configured user-defined RT tolerance of the assigned RT (no crossed adducts flagged)
- Downstream RT assignment agrees with manual review: manual vs metScribeR-derived RTs showed a correlation of 0.99

## Limitations

- RT is not consistently transferable or predictable across different laboratory-specific or method-specific LC-MS conditions, so tolerances cannot be reused across methods
- For large submissions the initial computation is slow (~30 min for 12000 mzML files), so the tolerance choice should be settled before resubmitting; loading storage_object_initial.RDS restarts downstream steps without repeating it
- No metScribeR version is stated; behavior of the tolerance parameters may differ across GitHub builds
- Adduct distinction assumes the instrument can confidently separate two signals; co-eluting adducts within tolerance will be confounded

## Evidence
