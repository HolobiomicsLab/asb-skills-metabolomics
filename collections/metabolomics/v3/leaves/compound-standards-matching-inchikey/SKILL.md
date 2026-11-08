---
name: compound-standards-matching-inchikey
description: Use when computing detected vs. correctly annotated compound rates by
  matching tool-reported identifications against a known standards list via exact
  InChIKey plus RT (<20 s) and mass (5 ppm) criteria.
when_to_use_negative:
- The dataset has no curated standards list with InChIKeys and reference RTs (e.g.,
  untargeted discovery samples) — correctness cannot be scored
- The input is already a validated feature/annotation table with confirmed identities,
  so re-matching would double-count
- The spectra are SWATH-DIA rather than DDA ESI+ standards-mixture data, where deconvolution
  and the benchmark thresholds (20 s RT, 5 ppm) were established
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: MetaboAnalystR
  role: 'Produces the identifications being matched: PerformDBSearchingBatch against
    MS2/MS1/RT reference libraries, then PerformResultsExport exports the search results
    as a data frame for InChIKey matching'
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MS-DIAL
  role: Baseline pipeline that deconvolves/detects MS2-bearing features (MS1 tolerance
    0.005 Da, MS2 tolerance 0.01 Da, minimum 50% peaks within group) whose MS-FINDER
    annotations are matched to standards
- name: MS-FINDER
  role: Batch-annotates MS-DIAL features (v3.52) to formulas/structures that are InChIKey-matched
    to the standards list
- name: MZmine
  role: Builds MSn feature lists (ADAP chromatogram builder at 0.005 Da or 10 ppm
    scan-to-scan; merged MS2) whose SIRIUS annotations are matched to standards
- name: SIRIUS
  role: Annotates MZmine features (v5.6.3, ZODIAC/CSI:FingerID/CANOPUS) for standards
    matching
- name: R
  role: Environment in which the exact InChIKey, RT (<20 s), and 5 ppm mass matching
    is scripted and the comparison table is generated
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:f70a076bce1d29d9da5f602a3bc28059d7694896bb7e5750f11cc6ad8a5b601c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MS-DIAL
  - MS-FINDER
  - MZmine
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# compound-standards-matching-inchikey

## Summary

Validates compound identifications from competing LC-MS/MS annotation pipelines (MetaboAnalystR 4.0 with/without DDA deconvolution, MS-DIAL/MS-FINDER, MZmine/SIRIUS) by exact InChIKey matching against a curated 526-compound standards list, applying retention-time deviation (<20 s) and mass tolerance (5 ppm) filters to compute correct-annotation rates.

## When to use

Use when computing detected vs. correctly annotated compound rates by matching tool-reported identifications against a known standards list via exact InChIKey plus RT (<20 s) and mass (5 ppm) criteria.

## When NOT to use

- The dataset has no curated standards list with InChIKeys and reference RTs (e.g., untargeted discovery samples) — correctness cannot be scored
- The input is already a validated feature/annotation table with confirmed identities, so re-matching would double-count
- The spectra are SWATH-DIA rather than DDA ESI+ standards-mixture data, where deconvolution and the benchmark thresholds (20 s RT, 5 ppm) were established

## Inputs

- Centroid-mode mzML raw spectra of the 1st standards mixture (DDA, ESI+, 526 standards) from Metabolomics Workbench accession ST002796
- Curated standards list with compound names, InChIKeys, retention times, and monoisotopic masses
- MetaboAnalystR identification table exported via PerformResultsExport (deconvoluted and non-deconvoluted runs)
- MS-FINDER export of identified formulas and structures for MS2-bearing MS-DIAL features
- SIRIUS (CSI:FingerID/CANOPUS) annotation of the exported MZmine feature lists

## Outputs

- table2_dda_esiplus_comparison.csv with detected and correctly annotated compound counts and rates per tool/condition
- Per-tool per-standard match table (InChIKey hit, RT deviation, ppm error)

## How to apply

Collect identifications from each pipeline: MetaboAnalystR 4.0 identifications exported via PerformResultsExport after PerformDBSearchingBatch (run once with PerformDDADeconvolution enabled and once disabled), MS-FINDER formulas/structures exported from MS-DIAL features (v4.9.22; MS1 tolerance 0.005 Da, MS2 tolerance 0.01 Da), and SIRIUS (v5.6.3; ZODIAC, CSI:FingerID, CANOPUS) annotations of MZmine (v3.2.8) feature lists. Normalize every identification to its full InChIKey and require exact InChIKey string equality with an entry in the 526-standard list for the 1st standards mixture (DDA, ESI+) from Metabolomics Workbench ST002796. Additionally enforce a retention-time deviation under 20 s and a mass tolerance of 5 ppm against each standard's reference values; only hits passing all three criteria count as correctly annotated. Compute, per tool/condition, the number and percentage of detected and correctly annotated compounds, and write the comparison as table2_dda_esiplus_comparison.csv. The rationale is that InChIKey matching removes ambiguity from tool-specific naming (synonyms, formula-only hits) while RT and ppm filters prevent false positives from isobaric coeluting features.

## Related tools

- **MetaboAnalystR** (Produces the identifications being matched: PerformDBSearchingBatch against MS2/MS1/RT reference libraries, then PerformResultsExport exports the search results as a data frame for InChIKey matching) — https://github.com/xia-lab/MetaboAnalystR
- **MS-DIAL** (Baseline pipeline that deconvolves/detects MS2-bearing features (MS1 tolerance 0.005 Da, MS2 tolerance 0.01 Da, minimum 50% peaks within group) whose MS-FINDER annotations are matched to standards)
- **MS-FINDER** (Batch-annotates MS-DIAL features (v3.52) to formulas/structures that are InChIKey-matched to the standards list)
- **MZmine** (Builds MSn feature lists (ADAP chromatogram builder at 0.005 Da or 10 ppm scan-to-scan; merged MS2) whose SIRIUS annotations are matched to standards)
- **SIRIUS** (Annotates MZmine features (v5.6.3, ZODIAC/CSI:FingerID/CANOPUS) for standards matching)
- **R** (Environment in which the exact InChIKey, RT (<20 s), and 5 ppm mass matching is scripted and the comparison table is generated) — https://github.com/xia-lab/MetaboAnalystR

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Deconvoluted MetaboAnalystR run must yield 159/526 correctly annotated compounds (39.2%) versus 146 (36.0%) without deconvolution, exceeding MS-DIAL/MS-FINDER (82, 20.2%) and MZmine/SIRIUS (77, 19.0%)
- Every accepted match must satisfy all three criteria simultaneously: exact InChIKey equality, RT deviation <20 s, and mass error within 5 ppm
- Relaxing any criterion (partial InChIKey blocks, wider ppm or RT windows) should inflate counts — verify counts are stable under the exact thresholds
- The exported CSV must contain, for each tool/condition, both a detected-compound rate and a correctly annotated-compound rate
- Each tool's identification list should map to the 526-standard universe with no duplicate standards counted twice

## Limitations

- Requires a curated standards list with InChIKeys, reference RTs, and masses; the approach cannot score correctness for compounds absent from the standards mixture
- RT matching (20 s window) assumes identical chromatographic conditions; retention-time drift between the tool run and the standards list causes false negatives
- The article notes its built-in MS2 reference libraries are 'compiled from various instruments and experimental conditions, resulting in potentially varying qualities', which propagates into identification and thus match rates
- Benchmark results are version-sensitive (MS-DIAL 4.9.22, MS-FINDER 3.52, MZmine 3.2.8, SIRIUS 5.6.3) and specific to this DDA ESI+ standards mixture

## Evidence

- [intro] Matching identifications to reference RT/mass values: "compound identifications typically involve matching m/z values and retention times of MS1 features, as well as their associated MS2 patterns against reference databases"
- [methods] Identifications come from a batch database search: "The database searching is performed via SQLite query and can be achieved with the function, PerformDBSearchingBatch."
- [methods] Search results are exported as a data frame for downstream matching: "The database search results can be exported as a data frame table using the PerformResultsExport function."
- [readme] Benchmark context: identification-rate improvements justify the correct-annotation metric: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
