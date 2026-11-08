---
name: lipidomics-peak-table-filtering
description: Use when you have a chromatographic peak table from Compound Discoverer
  (v3.3 .cdResult or .csv) plus lipid identifications and want to remove adduct peaks,
  in-source fragments, dimers, and redundant unidentified features correlated with
  identified lipids before downstream statistics.
when_to_use_negative:
- Input is a raw spectral file (.mgf, .mzXML, .raw) rather than a chromatographic
  peak table — use Spectrum Searcher for spectral matching instead of peak-table filtering.
- You need in-silico spectral library generation or fragment annotation — those are
  Library Generator and Lipid Spectral Annotator roles, not peak-table filtering.
- The dataset lacks QC replicates or a sufficient number of non-QC, non-blank samples
  (the correlation filter requires features quantified in ≥10 such samples), so the
  ≥0.99 correlation and %RSD criteria cannot be evaluated.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2
  role: Hosts the Peak Finder module that performs pairwise correlation filtering
    between unidentified and identified features and applies the 7 post-identification
    filter criteria; rewritten in C# .NET 6 for speed and Thermo RawFileReader compatibility.
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Spectrum Searcher
  role: Provides the MS2 and MS3+ spectral matches whose lipid identifications serve
    as the reference (identified) features against which unidentified co-eluting features
    are correlated and filtered.
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Degreaser
  role: Downstream QC module that validates filtered lipid identifications using six
    evidence checks (ECN vs RT, dilution series linear range, biosample quantitation
    range, spectral match validation, replicate QC %RSD with run order, and Compound
    Discoverer Peak Quality Factors).
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Upstream peak-detection software producing the .cdResult (v3.3) or .csv peak
    table that Peak Finder consumes as input; also supplies Peak Quality Factors used
    by Degreaser.
  repo: https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:63052e9e77af515b322b92dc55b77493a11174626ad690d1cb8e971d89dbe1f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  - LipiDex 2 Degreaser
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# lipidomics-peak-table-filtering

## Summary

Filter LC-MS lipidomics peak tables with LipiDex 2's Peak Finder correlation-based filtering and post-processing criteria to remove adducts, dimers, in-source fragments, and spurious co-eluting features, boosting confidence in remaining lipid identifications and quantitation.

## When to use

Use when you have a chromatographic peak table from Compound Discoverer (v3.3 .cdResult or .csv) plus lipid identifications and want to remove adduct peaks, in-source fragments, dimers, and redundant unidentified features correlated with identified lipids before downstream statistics.

## When NOT to use

- Input is a raw spectral file (.mgf, .mzXML, .raw) rather than a chromatographic peak table — use Spectrum Searcher for spectral matching instead of peak-table filtering.
- You need in-silico spectral library generation or fragment annotation — those are Library Generator and Lipid Spectral Annotator roles, not peak-table filtering.
- The dataset lacks QC replicates or a sufficient number of non-QC, non-blank samples (the correlation filter requires features quantified in ≥10 such samples), so the ≥0.99 correlation and %RSD criteria cannot be evaluated.

## Inputs

- Chromatographic peak table from Compound Discoverer (3.3 .cdResult file, or .csv intermediate)
- Lipid identifications from LipiDex 2 Spectrum Searcher (MS2 and MS3+ spectral matches)
- FWHM retention-time values for identified lipid features
- Per-feature intensity profiles across non-QC, non-blank biosamples
- QC replicate sample intensities (for %RSD criterion)

## Outputs

- Filtered peak table with adducts, dimers, in-source fragments, and correlated spurious unidentified features removed (e.g. filtered_peak_features.csv)
- Reduced counts of unidentified features and lipid identifications (e.g. 14,826 → 13,898 unidentified; 779 → 719 identified)
- Identifications downgraded to sum composition or plasmanyl when required-fragment or plasmenyl evidence is insufficient
- Results formatted for direct compatibility with Metaboanalyst

## How to apply

Load the peak table into LipiDex 2 Peak Finder (accepting direct Compound Discoverer 3.3 .cdResult files, avoiding intermediate .csv conversion) along with the lipid identifications from the Spectrum Searcher spectral matching step. Run the pairwise correlation filter, which compares unidentified features against identified lipid features that co-elute within an RT window of FWHM ± 0.5×FWHM around the target apex; an unidentified feature is removed only if it (1) lies within that RT window, (2) is quantified in at least 10 non-QC, non-blank samples, and (3) has a Pearson correlation ≥ 0.99 (default cutoff) with the identified feature's intensity profile — this removes adducts, dimers, and other spurious features correlated with the primary lipid peak. Then apply the 7 post-processing filter criteria (e.g., features with >30% RSD in QC samples are flagged for removal; RT modeling flags identifications with large deviation from predicted RT; required-fragment logic downgrades identifications to sum composition or plasmanyl when specific evidence is absent). On the reprocessed Nile rat plasma dataset (MSV000091033), this pipeline reduced 14,826 unidentified features and 779 lipid identifications to 13,898 unidentified and 719 identified features. Finally, export the filtered feature table and optionally route it to Degreaser QC, which applies six additional evidence checks (ECN vs. RT modeling, dilution series linear dynamic range, biosample quantitations within linear range, spectral match validation for all matches in the lipid feature, replicate QC %RSD with run order, and Compound Discoverer Peak Quality Factors).

## Related tools

- **LipiDex 2** (Hosts the Peak Finder module that performs pairwise correlation filtering between unidentified and identified features and applies the 7 post-identification filter criteria; rewritten in C# .NET 6 for speed and Thermo RawFileReader compatibility.) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Spectrum Searcher** (Provides the MS2 and MS3+ spectral matches whose lipid identifications serve as the reference (identified) features against which unidentified co-eluting features are correlated and filtered.) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Degreaser** (Downstream QC module that validates filtered lipid identifications using six evidence checks (ECN vs RT, dilution series linear range, biosample quantitation range, spectral match validation, replicate QC %RSD with run order, and Compound Discoverer Peak Quality Factors).) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Upstream peak-detection software producing the .cdResult (v3.3) or .csv peak table that Peak Finder consumes as input; also supplies Peak Quality Factors used by Degreaser.) — https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html

## Examples

```
Run LipiDex2.exe → Peak Finder → select your Compound Discoverer 3.3 .cdResult peak table, set correlation cutoff 0.99, minimum samples 10, RT window FWHM ± 0.5×FWHM, then export filtered_peak_features.csv
```

## Evaluation signals

- Count check: after filtering, unidentified-feature and lipid-identification counts should drop relative to pre-filter counts; on the MSV000091033 Nile rat plasma reprocess, expect 14,826 → 13,898 unidentified and 779 → 719 identified.
- Correlation-window check: confirm every removed unidentified feature co-eluted within FWHM ± 0.5×FWHM of an identified feature and had Pearson correlation ≥ 0.99 across ≥10 non-QC, non-blank samples.
- Adduct/dimer check: filtered-out features should be attributable to adducts, dimers, or in-source fragments correlated with primary lipid peaks, not to independently varying lipids.
- Downgrade check: lipids missing a required diagnostic fragment should be downgraded to sum composition, and insufficient plasmenyl evidence should downgrade to plasmanyl — verify these downgrades appear in the output table.
- Metaboanalyst compatibility: export the filtered table and confirm it loads directly into Metaboanalyst without manual reformatting.

## Limitations

- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table (direct reading avoids .csv conversion); other formats need an intermediate conversion step.
- Legacy LipiDex 1 is no longer supported and accepts only .mgf/.mzXML MS/MS files plus Compound Discoverer or mzMine 2 peak tables — older pipelines cannot use the .cdResult direct path.
- Correlation filtering depends on the default 0.99 Pearson cutoff and the ≥10-sample minimum; datasets with fewer biosamples or highly variable QC replicates (>30% RSD) risk losing genuine features.
- LipiDex 2 Wiki documentation is a work in progress ('WiP'), which may complicate parameter setup.

## Evidence

- [results] An unidentified feature is filtered only if it satisfies three criteria: it lies within the FWHM ± 0.5*FWHM window of the identified feature, appears in at least 10 non-QC, non-blank samples, and has a high default correlation value of 0.99.: [evidence span withheld — anchor status: not_found]
- [results] Applying Peak Finder filtering reduced 14826 unidentified features and 779 lipid identifications to 13898 unidentified features and 719 identified features by removing adducts, dimers, and other spurious features.: [evidence span withheld — anchor status: not_found]
