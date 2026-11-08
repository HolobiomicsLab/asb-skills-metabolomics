---
name: grouped-barchart-reproduction-from-fraction-tables
description: Use when per-dataset fraction values for several adduct classes (Na+,
  K+, 13C, DHB or CHCA matrix adducts) must be assembled across multiple MALDI-MSI
  datasets and rendered as a grouped bar chart to compare platforms.
when_to_use_negative:
- The goal is raw discovery of adduct pairs rather than re-summarizing already-computed
  fractions — use massdiff/adductMatch/corrPairsMSI directly instead.
- Only a single dataset is available; the grouped comparison across platforms (atmospheric-pressure
  vs high-vacuum) is the whole point of this skill.
- Input is already a polished Figure 3 image or pre-computed fraction table with no
  per-dataset counts to reaggregate.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mass2adduct
  role: Upstream R package whose massdiff, adductMatch, topAdducts and corrPairsMSI
    functions supply the per-dataset significant adduct-peak counts that are converted
    into fractions for the grouped bar chart
  repo: github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment for mass2adduct and for assembling and plotting the grouped
    bar chart from fraction tables
- name: SCiLS
  role: Source of the plain-text CSV MSI peak exports that mass2adduct ingests to
    produce the per-dataset counts feeding the chart
- name: MSiReader
  role: Alternative source of the CSV MSI peak exports that mass2adduct ingests for
    this analysis
- name: Cardinal
  role: Optional pre-processing path; MSProcessedImagingExperiment/MSContinuousImagingExperiment
    objects can be converted with cardinal2msimat() into the msimat format used upstream
    of the chart
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:144bc1c460ec6afaa733a1240e4af542ae4c67a6d48a08e2cee9032b870ed0bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# grouped-barchart-reproduction-from-fraction-tables

## Summary

Assembles per-dataset adduct-fraction counts (e.g. Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + CHCA + Na]+) into a grouped bar chart that reproduces Figure 3 of Janda et al. 2021, contrasting atmospheric-pressure (~31% adduct-driven peaks) vs high-vacuum (~6%) MALDI-MSI platforms.

## When to use

Use when per-dataset fraction values for several adduct classes (Na+, K+, 13C, DHB or CHCA matrix adducts) must be assembled across multiple MALDI-MSI datasets and rendered as a grouped bar chart to compare platforms.

## When NOT to use

- The goal is raw discovery of adduct pairs rather than re-summarizing already-computed fractions — use massdiff/adductMatch/corrPairsMSI directly instead.
- Only a single dataset is available; the grouped comparison across platforms (atmospheric-pressure vs high-vacuum) is the whole point of this skill.
- Input is already a polished Figure 3 image or pre-computed fraction table with no per-dataset counts to reaggregate.

## Inputs

- Per-dataset, per-adduct-class peak counts and total peak counts for 20 MALDI-MSI datasets (CSV/SCiLS or MSiReader exports, or msimat objects from mass2adduct)
- massdiff/adductMatch/corrPairsMSI outputs listing significant, co-localized adduct pairs per dataset
- Metadata mapping each dataset to its instrument platform (AP-SMALDI10/Orbitrap, SmartBeam-II/MRMS, MALDI2/QTOF)

## Outputs

- Grouped bar chart reproducing Figure 3: adduct fraction per adduct class, grouped by dataset/instrument platform
- Summary table of per-dataset adduct fractions for Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + CHCA + Na]+

## How to apply

Compute, for each MALDI-MSI dataset, the count of significant co-localized adduct peaks per class (Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + CHCA + Na]+) and divide by the total peak count in that dataset to obtain an adduct fraction. Assemble these fractions into a per-dataset, per-adduct summary table, then render as a grouped bar chart with one group per dataset/platform, highlighting the contrast between AP-SMALDI10/Orbitrap atmospheric-pressure datasets (~31% of peaks are matrix adducts) and high-vacuum SmartBeam-II/MRMS and MALDI2/QTOF platforms (~6%). This reproduces the Figure 3 pattern reported by Janda et al. using mass2adduct outputs (massdiff/adductMatch/corrPairsMSI). Validate that the reproduced per-dataset fractions match the published values, i.e. DHB-matrix datasets show the ~31% vs ~6% platform contrast and Na+/K+/13C/[M + (DHB-H2O) + H]+/[M + CHCA + Na]+ bars agree with the published values.

## Related tools

- **mass2adduct** (Upstream R package whose massdiff, adductMatch, topAdducts and corrPairsMSI functions supply the per-dataset significant adduct-peak counts that are converted into fractions for the grouped bar chart) — github.com/kbseah/mass2adduct
- **R** (Runtime environment for mass2adduct and for assembling and plotting the grouped bar chart from fraction tables)
- **SCiLS** (Source of the plain-text CSV MSI peak exports that mass2adduct ingests to produce the per-dataset counts feeding the chart)
- **MSiReader** (Alternative source of the CSV MSI peak exports that mass2adduct ingests for this analysis)
- **Cardinal** (Optional pre-processing path; MSProcessedImagingExperiment/MSContinuousImagingExperiment objects can be converted with cardinal2msimat() into the msimat format used upstream of the chart)

## Examples

```
R> d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist)); d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1); d.corr <- corrPairsMSI(d, d.diff.DHBH2O)
```

## Evaluation signals

- Reproduced per-dataset fractions must match Figure 3 of Janda et al. 2021, i.e. DHB-matrix datasets show the reported ~31% (AP-SMALDI/Orbitrap) vs ~6% (SmartBeam-II/MRMS and MALDI2/QTOF) contrast.
- Each per-dataset adduct fraction equals (significant matched peaks for that class) / (total peak count in that dataset); spot-check the division.
- All five adduct classes (Na+, K+, 13C, [M + (DHB-H2O) + H]+, [M + CHCA + Na]+) appear as bars for every dataset group.
- Only pairs surviving spatial co-localization (corrPairsMSI, Pearson correlation, default p=0.05 with Bonferroni correction) should be counted in the numerators.
- Platform-level grouping must be consistent with dataset metadata (atmospheric-pressure vs high-vacuum MALDI sources).

## Limitations

- Fraction values depend on instrument mass precision; massdiff histogram bin width (e.g. 0.01 in the README example) must match the known mass accuracy of each instrument before counting.
- Correlation-based adduct assignment (corrPairsMSI) is memory-limited for very large datasets and needs corrPairsMSIchunks with an explicit mem.limit to run serially in chunks.
- Adduct-matching via adductMatch reports only known adducts from the built-in adducts/adducts2 reference tables; unrecognized but abundant mass differences require topAdducts for discovery and will not appear in the predefined adduct-class bars.
- Quantile values reported by adductMatch are usually high because most mass differences have zero to few counts; they must not be misread as adduct abundance.

## Evidence

- [abstract] up to one third: "reveals a major influence of so far unrecognized metabolite–matrix adducts on total peak counts (up to one third)"
- [readme] adductMatch quantile note: "Note that quantile values will usually be quite high because the majority of mass differences have zero to few counts."
- [readme] corrPairsMSI significance cutoff: [evidence span withheld — anchor status: external_doc]
- [methods] corrPairsMSI correlation method: "This performs a correlation test (by default two-tailed with Pearson's method) on each pair of peaks in the massdiff object."
- [readme] memory chunking for large datasets: "To avoid this, use `corrPairsMSIchunks` instead of `corrPairsMSI`, and specify the maximum available memory with `mem.limit=N`"
- [readme] bin-width guidance: [evidence span withheld — anchor status: external_doc]
- [methods] adducts reference tables: "There are two built-in data sets `adducts` and `adducts2` (shorter), which list biologically-relevant chemical species that might occur in biological samples."
- [methods] input data format for msimat: "This should be exported as plain-text CSV files from standard MSI software such as SCiLS or MSiReader."
- [methods] Cardinal conversion path: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format with the `cardinal2msimat()` function"
- [methods] topAdducts complementary discovery: "`topAdducts` ranks mass differences by their occurrences, and reports them in descending order, as well as matches to known adducts, if any"
