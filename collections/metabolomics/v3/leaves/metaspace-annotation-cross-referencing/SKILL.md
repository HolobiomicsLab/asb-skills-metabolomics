---
name: metaspace-annotation-cross-referencing
description: Use when annotating MALDI-MSI peaks by exact m/z (e.g. via METASPACE)
  and you need to test whether some annotated ions are actually matrix adducts like
  [M + (DHB-H2O) + H]+ or [M + DHB + Na]+ of other peaks in the same spectrum.
when_to_use_negative:
- When the input is a raw, un-binned imzML file that has not been peak-picked or peak-binned
  — mass2adduct operates on peak lists (e.g. Cardinal MSProcessedImagingExperiment
  after peakBin())
- When your annotation database already incorporates matrix adducts explicitly and
  you are not relying on exact mass alone — this skill targets the gap where only
  exact m/z is used for annotation
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: R package used to build the parent/adduct candidate list (massdiff, adductMatch,
    topAdducts), estimate spatial correlation between parent and adduct ions (corrPairsMSI/corrPairsMSIchunks),
    and match annotated m/z values against known matrix adduct masses
  repo: https://github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment for executing the mass2adduct workflow (library(mass2adduct))
- name: METASPACE
  role: MSI annotation platform whose metabolite/lipid annotation results are cross-referenced
    against the computed adduct list to detect false positives
- name: Cardinal
  role: Optional upstream R package for peak-binning MSI data before conversion to
    mass2adduct's msimat format via cardinal2msimat()
  repo: http://cardinalmsi.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:f3597d93745b0b350869314fdf2bd3d36e6cebba13d3eada7e99ea90bb6ab732
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbseah/mass2adduct
  tools:
  - Cardinal
  - METASPACE
  - R
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# metaspace-annotation-cross-referencing

## Summary

Cross-reference metabolite annotations from a platform such as METASPACE against a putative adduct-mass list (e.g. DHB matrix adducts) to flag ions whose 'annotation' is better explained as an adduct of a co-observed parent peak, exposing mass-based false positives.

## When to use

Use when annotating MALDI-MSI peaks by exact m/z (e.g. via METASPACE) and you need to test whether some annotated ions are actually matrix adducts like [M + (DHB-H2O) + H]+ or [M + DHB + Na]+ of other peaks in the same spectrum.

## When NOT to use

- When the input is a raw, un-binned imzML file that has not been peak-picked or peak-binned — mass2adduct operates on peak lists (e.g. Cardinal MSProcessedImagingExperiment after peakBin())
- When your annotation database already incorporates matrix adducts explicitly and you are not relying on exact mass alone — this skill targets the gap where only exact m/z is used for annotation

## Inputs

- MSI peak list (m/z values) exported from MSI software such as SCiLS or MSiReader as plain-text CSV, or an R `msimat`/`massdiff` object from mass2adduct
- METASPACE (or equivalent) annotation table for a MALDI-MSI dataset, queried against a database such as LIPID_MAPS with an FDR threshold
- Adduct reference table (mass2adduct's built-in `adducts`/`adducts2`, or a custom data.frame with columns name/formula/mass) including matrix adducts like [M + (DHB-H2O) + H]+ and [M + DHB + Na]+

## Outputs

- Table of annotated m/z values matched against putative adduct pairs, showing for each case the platform annotation and the computed parent+adduct explanation
- Flagged false-positive annotations (e.g. m/z 878.591 annotated as PS(21:0/22:6) but matching [PC(P-16:0/18:2) + (DHB-H2O) + H]+)
- massdiff object with correlated parent/adduct ion pairs and Bonferroni-corrected p-values (corrPairsMSI output)

## How to apply

Build a candidate parent/adduct peak list from the observed ion masses (e.g. via the mass2adduct package's massdiff/adductMatch workflow) using the built-in adducts/adducts2 reference tables plus matrix-adduct entries such as [M + (DHB-H2O) + H]+ (m/z shift 136.01600) and [M + DHB + Na]+, with a search window around 1 mDa. Fetch the annotation table from the MSI annotation platform (e.g. METASPACE, queried against LIPID_MAPS at a specified FDR cutoff such as < 20%). For each annotated m/z, check whether a co-observed peak with mass exactly matching a known adduct difference of that annotation exists in the same dataset, and whether the adduct pair is spatially correlated (Pearson correlation on pixel intensities, Bonferroni-corrected p=0.05 via corrPairsMSI). An annotated ion whose m/z matches an adduct of another peak — but whose putative formula differs from the parent by the adduct formula (e.g. differing by C7H4O3 for a DHB-H2O adduct) — is flagged as a probable false-positive annotation. Validate by confirming concrete case-study pairs (e.g. m/z 878.591 annotated as PS(21:0/22:6) but matching [PC(P-16:0/18:2) + (DHB-H2O) + H]+ at m/z 742.575 [M+H]+).

## Related tools

- **mass2adduct** (R package used to build the parent/adduct candidate list (massdiff, adductMatch, topAdducts), estimate spatial correlation between parent and adduct ions (corrPairsMSI/corrPairsMSIchunks), and match annotated m/z values against known matrix adduct masses) — https://github.com/kbseah/mass2adduct
- **R** (Runtime environment for executing the mass2adduct workflow (library(mass2adduct)))
- **METASPACE** (MSI annotation platform whose metabolite/lipid annotation results are cross-referenced against the computed adduct list to detect false positives)
- **Cardinal** (Optional upstream R package for peak-binning MSI data before conversion to mass2adduct's msimat format via cardinal2msimat()) — http://cardinalmsi.org/

## Examples

```
Rscript -e 'library(mass2adduct); d <- msimat("msi.csv", sep=";"); d.diff <- massdiff(d); d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist))'
```

## Evaluation signals

- The comparison table must contain the concrete case-study pair: m/z 878.591 annotated as PS(21:0/22:6) and simultaneously matched as [PC(P-16:0/18:2) + (DHB-H2O) + H]+ of m/z 742.575 [M+H]+
- The molecular formula difference between the false-positive annotation and its parent must equal the adduct formula (e.g. C7H4O3 for DHB-H2O), not an arbitrary formula change
- Parent–adduct ion pairs retained in the final output should pass a spatial correlation test (Pearson, Bonferroni-corrected p ≤ 0.05 via corrPairsMSI)
- Mass differences between flagged parent/adduct pairs should fall within the declared search window (e.g. ~1 mDa or ppm-based tolerance around 136.01600 for DHB-H2O)
- Histogram binning of massdiff values must use an instrument-appropriate bin width (e.g. 0.01) before adductMatch is applied; un-binned raw differences should not be used for matching

## Limitations

- Mass differences computed directly from peak lists are 'misleadingly precise' — they must be binned into a histogram with a user-specified bin width tied to the instrument's mass precision before matching to known adducts
- For very large MSI CSV files (on the order of several Gb), direct reading into R may exceed available memory; use the triplet format via msimunging.pl or corrPairsMSIchunks with mem.limit
- Adduct-based matches are putative assignments, not confirmed identifications; quantile statistics from adductMatch are usually high because most mass differences have few counts
- Only known adducts in the reference list are matched by adductMatch — novel adducts require complementary screening with topAdducts

## Evidence

- [abstract] Flag annotated ions that match a matrix adduct of a co-observed parent peak rather than a genuine metabolite signal: "Metabolites ranging from lipids to amines and amino acids form matrix adducts with the commonly used 2,5-dihydroxybenzoic acid (DHB) matrix like [M + (DHB-H 2 O) + H] + and [M + DHB + Na] +"
- [discussion] The purpose of the cross-referencing is to prevent false annotations in mass-based annotation workflows: "including metabolite–matrix adducts into database annotations can reduce the number of unannotated peaks and, on the other hand, will prevent possible false annotations"
- [methods] Build the adduct candidate list from observed peaks and known adduct masses with mass2adduct: "Take all possible pairs of masses and calculate the mass difference for each pair. These mass differences represent potential molecular adducts."
- [readme] Match mass differences to known adducts via histogram binning: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] Verify parent/adduct pairs are spatially correlated before accepting an adduct explanation: [evidence span withheld — anchor status: external_doc]
- [methods] Annotate peaks corresponding to parent and adduct ions directly on the mass spectrum: "You can annotate the original mass spectrum using a massdiff object, to mark peaks corresponding to parent ions and adduct ions in different colors."
