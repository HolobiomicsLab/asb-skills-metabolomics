---
name: lcms-feature-table-analysis
description: Use when processing LC/MS feature tables from untargeted metabolomics
  where features must be grouped into compound spectra and annotated with ion species
  to cut feature count before statistical analysis or compound identification.
when_to_use_negative:
- When the input is already a feature table annotated at the compound level (no need
  to re-derive grouping rules)
- When only MS/MS spectral matching is needed and no LC feature grouping is required
- When the user has a single-polarity dataset and cannot tolerate the added complexity
  of cross-polarity rule overrides
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Primary Bioconductor package implementing compound spectra extraction, isotope/adduct
    peak annotation, and molecular mass proposal via a dynamic rule table
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream peak detection and alignment framework whose feature lists CAMERA
    consumes as input
- name: R
  role: Implementation language in which CAMERA is distributed via the Bioconductor
    repository
  repo: https://www.bioconductor.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:7918000aaf7db8582fed79ad11a5596bd6e6644a751ca2f241c061832ec17068
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lcms-feature-table-analysis

## Summary

Group LC/MS features derived from the same analyte and annotate ion species (adducts, isotopes, neutral losses) in feature tables using CAMERA's dynamic rule set, reducing feature-list redundancy and revealing quasi-molecular ions for downstream identification.

## When to use

Use when processing LC/MS feature tables from untargeted metabolomics where features must be grouped into compound spectra and annotated with ion species to cut feature count before statistical analysis or compound identification.

## When NOT to use

- When the input is already a feature table annotated at the compound level (no need to re-derive grouping rules)
- When only MS/MS spectral matching is needed and no LC feature grouping is required
- When the user has a single-polarity dataset and cannot tolerate the added complexity of cross-polarity rule overrides

## Inputs

- LC/MS feature table from peak detection and alignment (e.g. XCMS peak list or xsAnnotate object)
- Grouped peak data with retention time, m/z, and per-sample intensities
- Optional ion species/adduct rule list or user-defined neutral losses
- Optional paired positive/negative polarity data files

## Outputs

- Compound spectra (groups of related features) with quasi-molecular ion designation
- Annotated peak table with ion species assignments for each feature
- Molecular mass estimates and adduct/neutral-loss/cluster-ion annotations
- Reduced feature list for downstream statistical analysis

## How to apply

Starting from a feature table (e.g. an xsAnnotate object in CAMERA built on XCMS peak lists), first group related features using retention time, m/z difference, intensity correlation (calcCiS/calcCaS) and peak-shape methods (calcPC.lpc/calcPC.hcs) to form compound spectra. Then generate the dynamic annotation rule set via generateRules(), computing for each ion species its mass difference to the molecular mass, its charge, and molecule count. Match observed within-spectrum m/z differences to these rules with findAdducts(), using ppm and mzabs tolerance parameters to score candidates; do NOT rely on ad-hoc heuristics like assuming the most intense feature is [M+H]+. For data acquired in both polarities, combine spectral information across opposite ion modes with a cross-polarity rule table containing only common and trusted combinations, since these can override single-polarity annotations. Evaluate annotation completeness against CAMERA's reported quasi-molecular ion annotations, and restrict compounds to retention times above thresholds like 45 s to exclude early-eluting contaminants.

## Related tools

- **CAMERA** (Primary Bioconductor package implementing compound spectra extraction, isotope/adduct peak annotation, and molecular mass proposal via a dynamic rule table) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream peak detection and alignment framework whose feature lists CAMERA consumes as input)
- **R** (Implementation language in which CAMERA is distributed via the Bioconductor repository) — https://www.bioconductor.org

## Examples

```
library(CAMERA); xa <- xsAnnotate(xs); xa <- groupFWHM(xa); xa <- findAdducts(xa, polarity='positive', ppm=5, mzabs=0.01)
```

## Evaluation signals

- Quasi-molecular ion annotations recovered ~89.7% (positive mode) and 90.3% (negative mode) of annotatable compounds in validation
- Feature list size should shrink relative to the raw aligned peak table (prior work reported 50–86% reductions)
- Molecular mass estimates should be reproducible across replicate injections without detector saturation (avoid m/z and RT centroid shifts)
- Cross-polarity annotations should only be trusted when supported by common, trusted combinations of ion species
- Compounds below the retention-time cutoff (e.g. 45 s) must be excluded before grouping

## Limitations

- Cannot annotate compounds whose spectra contain only the molecular ion and fragment ions unless the neutral loss is explicitly added to the rule set
- Missing annotations occur when compound spectra contain too few explained features or when several hypotheses share identical precedence scores
- Detector saturation in high-concentration samples causes m/z and retention time centroid shifts that break grouping accuracy

## Evidence
