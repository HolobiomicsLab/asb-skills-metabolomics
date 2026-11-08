---
name: highly-connected-subgraph-community-detection
description: Use when a CAMERA xsAnnotate pseudospectrum from findIsotopes/groupFWHM
  is suspected to mix features of co-eluting analytes and needs to be separated into
  distinct compound spectra.
when_to_use_negative:
- The compound spectrum already contains features of a single analyte with consistent
  isotope/adduct grouping — graph-based splitting would risk incorrectly severing
  true adduct or fragment edges.
- Input is raw centroided or profile data that has not been peak-picked, grouped (groupFWHM)
  and isotope-annotated (findIsotopes); the graph method operates on the CAMERA pseudospectrum
  structure, not raw peaks.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Bioconductor package whose groupCorr function orchestrates the graph construction
    and community-detection split of co-eluting pseudospectra
  repo: https://travis-ci.org/sneumann/CAMERA
- name: RBGL
  role: Provides calcPC.hcs, the highly connected subgraphs algorithm used when graphMethod='hcs'
- name: igraph
  role: Provides calcPC.lpc, the label-propagation clustering used when graphMethod='lpc'
- name: xcms
  role: Upstream peak detection and grouping producing the feature list that CAMERA
    annotates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:0d8fff2d1dc0c21fb00948197746d03fad42e499e899701776b3a94ed5f8d82e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Biobase
  - CAMERA
  - R
  - RBGL
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# highly-connected-subgraph-community-detection

## Summary

Split an LC/MS compound spectrum that may still contain features of two or more closely co-eluting compounds into separate pseudospectra by detecting communities in a weighted feature-relationship graph using CAMERA's graph-based refinement (HCS or LPC clustering).

## When to use

Use when a CAMERA xsAnnotate pseudospectrum from findIsotopes/groupFWHM is suspected to mix features of co-eluting analytes and needs to be separated into distinct compound spectra.

## When NOT to use

- The compound spectrum already contains features of a single analyte with consistent isotope/adduct grouping — graph-based splitting would risk incorrectly severing true adduct or fragment edges.
- Input is raw centroided or profile data that has not been peak-picked, grouped (groupFWHM) and isotope-annotated (findIsotopes); the graph method operates on the CAMERA pseudospectrum structure, not raw peaks.

## Inputs

- xsAnnotate object from CAMERA (post groupFWHM and findIsotopes) saved as an R workspace file
- Extraction ion chromatograms (EICs) per peak across samples
- Cross-sample feature-intensity matrix for correlation scoring
- Parameters: cor_eic_th (correlation threshold), pval (significance cutoff), graphMethod ('hcs' or 'lpc'), calcIso, calcCaS

## Outputs

- Refined pseudospectra assignment (per-feature pseudospectrum / compound spectrum membership) inside the xsAnnotate object
- Annotated peak list exported as a tab-separated table, e.g. groupcorr_peaklist.tsv via getPeaklist

## How to apply

After initial retention-time-based grouping (groupFWHM) and isotope detection (findIsotopes), load the xsAnnotate object and call groupCorr, which internally computes extracted ion chromatograms (EICs) via getAllPeakEICs, scores cross-sample intensity-correlation edges with calcCiS (controlled by cor_eic_th and a p-value cutoff) and peak-shape correlation edges with calcCaS, then merges these with the isotope-relation edges into one combined weighted graph. Choose the community detection algorithm with graphMethod: 'hcs' (RBGL highly connected subgraphs, calcPC.hcs) or 'lpc' (igraph label propagation, calcPC.lpc); include isotope edges with calcIso=TRUE and peak-shape edges with calcCaS=TRUE. The split is expected to cut the weak edge between features of the two co-eluting analytes, yielding two separate pseudospectra; export the refined assignment with getPeaklist as a tab-separated table (e.g. groupcorr_peaklist.tsv).

## Related tools

- **CAMERA** (Bioconductor package whose groupCorr function orchestrates the graph construction and community-detection split of co-eluting pseudospectra) — https://travis-ci.org/sneumann/CAMERA
- **RBGL** (Provides calcPC.hcs, the highly connected subgraphs algorithm used when graphMethod='hcs')
- **igraph** (Provides calcPC.lpc, the label-propagation clustering used when graphMethod='lpc')
- **xcms** (Upstream peak detection and grouping producing the feature list that CAMERA annotates)

## Examples

```
Rscript -e "library(CAMERA); xset <- readRDS('xsAnnotate.RData'); ann <- groupCorr(xset, cor_eic_th=0.75, pval=0.05, graphMethod='hcs', calcIso=TRUE, calcCaS=TRUE); write.table(getPeaklist(ann), 'groupcorr_peaklist.tsv', sep='\t', quote=FALSE)"
```

## Evaluation signals

- On synthetic co-eluting two-compound data, the split must separate the two compounds into different pseudospectra (the weak inter-compound edge is cut).
- Every feature in the original xsAnnotate pseudospectrum appears in exactly one refined pseudospectrum (partition is complete and non-overlapping).
- Isotope edges (calcIso=TRUE) should remain within one pseudospectrum — isotope clusters of a single analyte must not be split apart.
- Exported groupcorr_peaklist.tsv must be a valid tab-separated table whose pseudospectrum column matches the in-object assignment from groupCorr.
- Changing graphMethod from 'hcs' to 'lpc' should yield consistent or near-identical splits on clean data; divergence indicates ambiguous edges worth inspecting.

## Limitations

- Annotations fail when the compound spectrum contains too few explained features or when several hypotheses receive the same precedence score, per the source article's missing-annotation analysis.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- graphMethod choice between 'hcs' and 'lpc' can influence splits; the article does not prescribe a universal default, so both should be assessed on ambiguous pseudospectra.
- The approach assumes features of one analyte are more strongly connected to each other than to features of the co-eluting neighbor; weak or noisy correlation edges may cause misassignment.

## Evidence

- [methods] HCS as the primary split algorithm: [evidence span withheld — anchor status: not_found]
- [methods] groupCorr call and parameters: [evidence span withheld — anchor status: not_found]
- [methods] Edge construction details: [evidence span withheld — anchor status: not_found]
- [results] Splitting quality expectation: [evidence span withheld — anchor status: not_found]
- [methods] Alternative LPC algorithm: [evidence span withheld — anchor status: not_found]
