---
name: ms2-scan-to-feature-association
description: Use when you have LC/MS data with detected molecular features (feature
  table from xcms) plus centroided MS/MS scans, and you need to associate each MS2
  spectrum with the correct feature to enable MS/MS spectral similarity or networking.
when_to_use_negative:
- Input MS data has not yet been processed into a molecular feature table (no xcms
  feature detection has been run)
- The analysis goal is purely MS1-level statistical comparison of features without
  any fragmentation-based annotation
- Data are profile-mode and not centroided, so MS2 peaks cannot be matched at ppm-level
  tolerance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Provides the FindMS2ScansModule ('Find MS2 scans' dialog) that associates
    MS2 spectra with features, and the network viewer/SaveNetworkModule (.graphML
    export)
  repo: https://github.com/mjhelf/Metaboseek
- name: MassTools
  role: R package used to compute pairwise spectrum similarities (makeEdges()/network1())
    from associated MS2 scans
  repo: https://github.com/mjhelf/MassTools
- name: igraph
  role: R package used to handle the molecular network graph (e.g. .graphML output)
  repo: https://github.com/igraph/rigraph
- name: xcms
  role: Underlying feature detection producing the feature table whose features are
    linked to MS2 scans
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:25ad94a50d6d93d22dcf1b14b2f13b690a652333f35036683ad5f61d27d9a829
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MassTools
  - Metaboseek
  - igraph
  - xcms
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# MS2 scan-to-feature association

## Summary

Links centroided MS/MS (MS2) scans to molecular features from an xcms-generated feature table in Metaboseek (FindMS2ScansModule), so that averaged fragmentation spectra can be used for downstream molecular networking of differential metabolites.

## When to use

Use when you have LC/MS data with detected molecular features (feature table from xcms) plus centroided MS/MS scans, and you need to associate each MS2 spectrum with the correct feature to enable MS/MS spectral similarity or networking.

## When NOT to use

- Input MS data has not yet been processed into a molecular feature table (no xcms feature detection has been run)
- The analysis goal is purely MS1-level statistical comparison of features without any fragmentation-based annotation
- Data are profile-mode and not centroided, so MS2 peaks cannot be matched at ppm-level tolerance

## Inputs

- Feature table of molecular features (xcms results loadable as a 'Feature Table' via xcms and MSnbase)
- Centroided MS/MS (MS2) scan data from LC/MS acquisition
- m/z tolerance setting in ppm
- Retention-time window setting

## Outputs

- Per-feature averaged MS2 spectra
- Feature-MS2 scan association table used for spectral similarity scoring
- Molecular network (e.g. exported as molecular_network.graphml via SaveNetworkModule)

## How to apply

Load the differential exo-metabolome feature table and associated centroided MS data files into Metaboseek via the Load module, then run the FindMS2ScansModule ('Find MS2 scans' dialog) using m/z tolerance in ppm and a retention-time window to match scans to features. Average MS2 scans per molecular feature before computing similarities. When building edges downstream with MassTools makeEdges()/network1(), exclude peaks below the 'Noise level in %' intensity threshold, set 'min. peaks' so similarity scores of 0 are returned when fewer peaks match, and optionally enable 'Use parent masses' so neutral losses contribute to peak matching; the cosine score is calculated between matched-peak intensity vectors. Apply a 'Cosine threshold' to keep only significant comparisons before exporting the network.

## Related tools

- **Metaboseek** (Provides the FindMS2ScansModule ('Find MS2 scans' dialog) that associates MS2 spectra with features, and the network viewer/SaveNetworkModule (.graphML export)) — https://github.com/mjhelf/Metaboseek
- **MassTools** (R package used to compute pairwise spectrum similarities (makeEdges()/network1()) from associated MS2 scans) — https://github.com/mjhelf/MassTools
- **igraph** (R package used to handle the molecular network graph (e.g. .graphML output)) — https://github.com/igraph/rigraph
- **xcms** (Underlying feature detection producing the feature table whose features are linked to MS2 scans) — https://github.com/sneumann/xcms

## Examples

```
Rscript -e 'devtools::install_github("mjhelf/Metaboseek"); devtools::install_github("mjhelf/MassTools"); library(MassTools); edges <- makeEdges(featureTable, ms2scans)'
```

## Evaluation signals

- Each differential hacl-1-enriched feature has at least one associated (averaged) MS2 spectrum
- Cosine scores are 0 for spectrum pairs with fewer than 'min. peaks' matched peaks, and low-intensity peaks below 'Noise level in %' are excluded from matching
- The resulting network resolves into the expected five major subnetworks SN1–SN5 (N-acyl GPEs; GLEA SN2–SN4; NAEs)
- N-acyl GPE subnetwork nodes carry the reported 10–100-fold hacl-1 exo-metabolome enrichment annotation
- Exported molecular_network.graphml loads correctly in a graph tool (igraph) with nodes and edges intact

## Limitations

- Similarity score becomes 0 when fewer than 'min. peaks' peaks match between two spectra, potentially dropping edges between sparse spectra
- Peaks below the 'Noise level in %' threshold are excluded, so weak fragment ions never contribute to cosine scores
- Metabolite annotations from this workflow are limited to confidence Level 3 (tentative structure or compound class assignments only); many features remain unidentified

## Evidence

- [methods] Noise level in %: "peaks at an intensity below a set percentage of the maximum peak intensity in a scan (`Noise level in %`) will be excluded"
- [methods] min. peaks: "The similarity score will be considered 0 if less than `min. peaks` match between two spectra"
- [methods] Feature Table loading: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [methods] Cosine threshold: "select a threshold for which comparisons to keep (above a given `Cosine threshold`)"
- [readme] Metaboseek architecture: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
