---
name: natural-product-structure-library-curation
description: Use when benchmarking or running spectral library searches against natural
  product structures and you must merge community structure databases (DNP, GNPS)
  into one searchable reference set with ground-truth annotations.
when_to_use_negative:
- Input is already a single well-defined community database such as PubChem (83M compounds)
  and no benchmark ground truth is needed - this skill is specifically for merging
  curated natural product sources with known annotations.
- You are dereplicating purely peptidic natural products with DEREPLICATOR/VarQuest
  on a single-structure-source database rather than building a merged multi-class
  reference set.
- The target metabolites fall outside natural product classes (polyketides, terpenes,
  benzenoids, alkaloids, flavonoids, etc.) covered by these community databases.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: DEREPLICATOR+
  role: Consumes the curated combined structure library to perform database search
    of mass spectra against diverse metabolite classes; library entries drive fragmentation
    graph construction.
  repo: http://mohimanilab.cbd.cmu.edu/software/
- name: GNPS
  role: Provides spectral library structures that are merged with DNP into the combined
    reference library; also hosts DEREPLICATOR+ as a web application.
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: DEREPLICATOR
  role: Earlier peptidic-natural-product-specific search tool whose library requirements
    and benchmark behavior (e.g. 17%/25% top-rank rates) provide a baseline for validating
    the curated library.
  repo: https://github.com/ablab/npdtools
- name: VarQuest
  role: Modification-tolerant identification of novel PNP variants; also relies on
    a curated chemical structure database of known natural products.
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:261806645ad1d553ab3f7fb3213e317974322f74d7f450c9bd05596c39c43ae3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  - VarQuest
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Natural product structure library curation

## Summary

Build a combined reference library of known natural product chemical structures (e.g. DNP plus GNPS library structures, ~86,665 compounds) so dereplication searches can match MS/MS spectra against candidate metabolites across multiple chemical classes.

## When to use

Use when benchmarking or running spectral library searches against natural product structures and you must merge community structure databases (DNP, GNPS) into one searchable reference set with ground-truth annotations.

## When NOT to use

- Input is already a single well-defined community database such as PubChem (83M compounds) and no benchmark ground truth is needed - this skill is specifically for merging curated natural product sources with known annotations.
- You are dereplicating purely peptidic natural products with DEREPLICATOR/VarQuest on a single-structure-source database rather than building a merged multi-class reference set.
- The target metabolites fall outside natural product classes (polyketides, terpenes, benzenoids, alkaloids, flavonoids, etc.) covered by these community databases.

## Inputs

- Dictionary of Natural Products (DNP) chemical structure database (~300k compounds)
- GNPS spectral library structures (library hits with chemical structures)
- Benchmark spectra with known ground-truth annotations (e.g. 5473 SpectraLibrary annotated MS/MS spectra)
- Chemical structure database of known natural products (required for Dereplicator, VarQuest and Dereplicator+ database search pipelines)

## Outputs

- Combined searchable structure library (~86,665 chemical structures from DNP + GNPS)
- Annotated structure entries with metabolite graph / fragmentation graph metadata for DEREPLICATOR+ searches
- Benchmark top-rank identification statistics (percentages and hit counts per tool and precursor tolerance)

## How to apply

Merge chemical structure sources such as the Dictionary of Natural Products (~300 thousand compounds) with the GNPS spectral library structures into a single combined library of known natural products. Curate entries so each structure carries the correct annotation/mass so that fragmenting the metabolite graph (DEREPLICATOR+ style C-N/C-O/C-C bond disconnection) yields a fragmentation graph for search. Validate the library by running the search pipeline over a benchmark spectrum set (e.g. 5473 annotated SpectraLibrary spectra) and confirming top-rank identification rates reproduce published values (e.g. DEREPLICATOR+ 34% at 0.5 Da tolerance, 44% at 0.02 Da tolerance). Keep tool versions and database composition fixed so benchmark numbers stay comparable across tolerance settings (0.5 vs 0.02 Da).

## Related tools

- **DEREPLICATOR+** (Consumes the curated combined structure library to perform database search of mass spectra against diverse metabolite classes; library entries drive fragmentation graph construction.) — http://mohimanilab.cbd.cmu.edu/software/
- **GNPS** (Provides spectral library structures that are merged with DNP into the combined reference library; also hosts DEREPLICATOR+ as a web application.) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **DEREPLICATOR** (Earlier peptidic-natural-product-specific search tool whose library requirements and benchmark behavior (e.g. 17%/25% top-rank rates) provide a baseline for validating the curated library.) — https://github.com/ablab/npdtools
- **VarQuest** (Modification-tolerant identification of novel PNP variants; also relies on a curated chemical structure database of known natural products.) — https://github.com/ablab/npdtools

## Examples

```
python dereplicator+.py --spectra spectra.mgf --database dnp_plus_gnps_structures --pmc 0.5 --output top_rank_percentages.csv
```

## Evaluation signals

- Top-rank identification percentages on the 5473-spectrum benchmark must match published values: DEREPLICATOR+ 34% (1878) at 0.5 Da and 44% (2414) at 0.02 Da; DEREPLICATOR-G 30% (1647) and 40% (2216); DEREPLICATOR 17% (964) and 25% (1362).
- Combined library size should equal the sum of DNP structures plus GNPS library structures (reported as 86,665 for the benchmark).
- Every structure entry must yield a valid metabolite/fragmentation graph when processed by DEREPLICATOR+ (all bonds C-N/C-O/C-C disconnected without errors).
- Search results must be reproducible at two precursor mass tolerances (0.5 Da and 0.02 Da), showing consistent relative ranking of tools and monotonic improvement at tighter tolerance.
- Ground-truth annotations for each benchmark spectrum must map unambiguously to a single library structure to score top-rank hits.

## Limitations

- The exact-formula-based dereplication that underlies many library searches often fails because the number of possible formulas rapidly increases with molecular mass and existing chemical databases contain many compounds with identical formulas.
- Similar to all existing metabolomics tools, DEREPLICATOR+ is not applicable to all classes of compounds, so a merged natural product library may still miss compounds outside its covered classes.
- The simple shared-peaks scoring used to match spectra against fragmentation graphs results in a high false-negative rate, so library quality alone does not guarantee identification; scoring improvements were not yet implemented in this work.
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, or VarQuest, so library curation must record database composition and date explicitly to keep results reproducible.

## Evidence

- [readme] Database search pipelines (Dereplicator, VarQuest and Dereplicator+) also require a chemical structure database of known natural products.: [evidence span withheld — anchor status: external_doc]
- [intro] ChEBI (440,000 compounds), Dictionary of Natural Products (≈300 thousand compounds), AntiMarin (≈60 thousand compounds): "ChEBI (440,000 compounds), Dictionary of Natural Products (≈300 thousand compounds), AntiMarin (≈60 thousand compounds)"
- [intro] Global Natural Products Social (GNPS) molecular networking project4 is a recent mass spectrometry data repository in the field of natural products: "Global Natural Products Social (GNPS) molecular networking project4 is a recent mass spectrometry data repository in the field of natural products"
- [intro] this approach often fails because the number of possible formulas rapidly increases with the molecular mass of metabolites and because existing chemical databases contain many compounds with: "this approach often fails because the number of possible formulas rapidly increases with the molecular mass of metabolites and because existing chemical databases contain many compounds with"
- [discussion] DEREPLICATOR+ is available as both a stand-alone tool (http://mohimanilab.cbd.cmu.edu/software/) and a web application: "DEREPLICATOR+ is available as both a stand-alone tool (http://mohimanilab.cbd.cmu.edu/software/) and a web application"
- [other] No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS: [evidence span withheld — anchor status: not_found]
