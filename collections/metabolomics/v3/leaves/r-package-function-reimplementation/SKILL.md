---
name: r-package-function-reimplementation
description: Use when a pipeline step depends on an R package function whose internals
  must be reproduced locally — e.g. annotating LC-MS/MS features against HMDB V5.0
  and LMSD with metID when metID itself cannot be installed or wrapped.
when_to_use_negative:
- The original package installs cleanly and the call can be made directly — reimplementing
  adds maintenance risk without benefit.
- The target function is not exposed in the package source or is undocumented; reproducing
  its internals from behavior alone is unreliable.
- The goal is to change annotation behavior (e.g. different databases or tolerances)
  rather than reproduce it; a reimplementation should faithfully copy, not diverge.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: MultiABLER
  role: Source package whose function wrapper is being reimplemented; provides the
    annotation workflow and the hmdb.lipids.massdb reference dataset.
  repo: https://github.com/holab-hku/MultiABLER
- name: metid
  role: Annotation engine the wrapper invokes to match feature m/z against HMDB V5.0
    and LMSD entries.
- name: devtools
  role: Used to install MultiABLER (and, transitively, its dependencies) when reproducing
    the wrapper's runtime environment.
- name: ProteoMM
  role: One of the dependencies installed first if MultiABLER's devtools install fails,
    per the README's installation-issues section.
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- name: limma
  role: Bioconductor dependency installed before retrying the MultiABLER install when
    issues arise.
  repo: https://www.bioconductor.org/packages/release/bioc/html/limma.html
- name: Tidyverse
  role: Dependency installed before retrying the MultiABLER install when issues arise.
  repo: https://www.tidyverse.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:9391d75c5796a297beb0ebbff82059d6d3d3625f02357e393404ce7528b934cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - MultiABLER
  - ProteoMM
  - R
  - Tidyverse
  - devtools
  - limma
  - metid
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4583333333333333
n_claims: 48
---

# r-package-function-reimplementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements a specific R function from an external package (here, the metID-based lipid/metabolite annotation used inside MultiABLER) by extracting the package source, reconstructing the wrapper's internal logic, and returning the same output structure without the heavyweight dependency.

## When to use

Use when a pipeline step depends on an R package function whose internals must be reproduced locally — e.g. annotating LC-MS/MS features against HMDB V5.0 and LMSD with metID when metID itself cannot be installed or wrapped.

## When NOT to use

- The original package installs cleanly and the call can be made directly — reimplementing adds maintenance risk without benefit.
- The target function is not exposed in the package source or is undocumented; reproducing its internals from behavior alone is unreliable.
- The goal is to change annotation behavior (e.g. different databases or tolerances) rather than reproduce it; a reimplementation should faithfully copy, not diverge.

## Inputs

- Processed LC-MS/MS feature table to annotate (m/z values per feature)
- HMDB V5.0 reference mass database (hmdb.lipids.massdb dataset from MultiABLER)

## Outputs

- Annotated feature table (data frame) with per-feature lipid vs metabolite label column matching the original wrapper's schema

## How to apply

Locate the target function in the package's repository (MultiABLER is on GitHub under holab-hku/MultiABLER) and read the wrapper R source to see which underlying package calls it makes. Reconstruct the call chain: for annotation, the wrapper loads the HMDB V5.0 reference mass database (hmdb.lipids.massdb, shipped as a MultiABLER dataset), matches each LC-MS/MS feature's m/z against HMDB V5.0 and LMSD entries with metID's annotation engine, and labels features overlapping LMSD as lipids and all other matched features as metabolites. Preserve the wrapper's input/output contract so callers see identical behavior; validation is expert review that the reimplementation returns a data frame with the same annotation-label column. Install dependencies as MultiABLER itself does — devtools::install_github with dependencies = TRUE — and, if that fails, first install ProteoMM, limma, and Tidyverse as the README prescribes.

## Related tools

- **MultiABLER** (Source package whose function wrapper is being reimplemented; provides the annotation workflow and the hmdb.lipids.massdb reference dataset.) — https://github.com/holab-hku/MultiABLER
- **metid** (Annotation engine the wrapper invokes to match feature m/z against HMDB V5.0 and LMSD entries.)
- **devtools** (Used to install MultiABLER (and, transitively, its dependencies) when reproducing the wrapper's runtime environment.)
- **ProteoMM** (One of the dependencies installed first if MultiABLER's devtools install fails, per the README's installation-issues section.) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- **limma** (Bioconductor dependency installed before retrying the MultiABLER install when issues arise.) — https://www.bioconductor.org/packages/release/bioc/html/limma.html
- **Tidyverse** (Dependency installed before retrying the MultiABLER install when issues arise.) — https://www.tidyverse.org

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- The reimplemented wrapper executes without error on the same LC-MS/MS feature table that the original annotated.
- The returned object is a data frame whose column structure matches the original wrapper's output, including the lipid/metabolite annotation label column.
- Features whose m/z overlaps LMSD entries are labeled lipids and the remaining matched features are labeled metabolites, mirroring the original labeling rule.
- The hmdb.lipids.massdb reference set is loaded from the MultiABLER dataset, not from an external copy, so annotation results are reproducible against the same HMDB V5.0 version.
- Independent expert review confirms the reimplementation reproduces the wrapper's behavior on a held-out feature table.

## Limitations

- Reimplementation is only as faithful as the source code you can access; if the package changes its internals (e.g. metID API changes), the reimplementation must be updated.
- The MultiABLER annotation path ties to a specific reference version — HMDB V5.0 / LMSD — and its labeling rule; using a different database version will change which features are called lipids.
- Install failures can arise from missing dependencies; the README prescribes installing ProteoMM, limma, and Tidyverse first, which may still leave unlisted transitive dependencies unresolved.

## Evidence

- [methods] the metabolomic and lipidomic peaks are annotated using metID against HMDB and LipidMaps, with LipidMaps entries labeled as lipids and the remaining features labeled as metabolites.: [evidence span withheld — anchor status: not_found]
