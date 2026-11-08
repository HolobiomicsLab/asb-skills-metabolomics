---
name: putative-annotation-table-processing
description: Use when an untargeted metabolomics dataset contains massive numbers
  of putative annotation entries (e.g., hundreds of candidate metabolites) that must
  be classified by origin (peptides, drugs, nutrients, plants, etc.) or reduced in
  redundancy.
when_to_use_negative:
- The dataset does not consist of putative annotation entries (e.g., raw spectra or
  already-confirmed metabolite identities) — Tagger is designed for classifying candidate/potential
  metabolites
- The goal is definitive biological interpretation rather than the 'preliminary insights
  into the biological system studied' that Tagger provides
- Feature intensity scoring is needed but no feature intensity and relevant metadata
  tables are available for TPMetrics to combine
tools:
- name: TurboPutative
  role: Web platform whose Tagger module classifies putative annotations by origin;
    REname, RowMerger, and TPMetrics handle redundancy, dataset reduction, and adduct/correlation
    scoring
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:d432673f4387f6e079f8190b9e17b6796214df4237aed689f59a57af63daeafb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# putative-annotation-table-processing

## Summary

Classify and manage the large volumes of putative annotation entries produced by untargeted metabolomics using the TurboPutative web platform, whose Tagger module assigns each potential metabolite an origin class and whose companion modules (REname, RowMerger, TPMetrics) handle redundancy and scoring.

## When to use

Use when an untargeted metabolomics dataset contains massive numbers of putative annotation entries (e.g., hundreds of candidate metabolites) that must be classified by origin (peptides, drugs, nutrients, plants, etc.) or reduced in redundancy.

## When NOT to use

- The dataset does not consist of putative annotation entries (e.g., raw spectra or already-confirmed metabolite identities) — Tagger is designed for classifying candidate/potential metabolites
- The goal is definitive biological interpretation rather than the 'preliminary insights into the biological system studied' that Tagger provides
- Feature intensity scoring is needed but no feature intensity and relevant metadata tables are available for TPMetrics to combine

## Inputs

- Input table of putative annotations from untargeted metabolomics (as provided by the TurboPutative web server, e.g., a working-example table of putative annotations)
- Optional feature intensity and metadata tables for TPMetrics scoring

## Outputs

- Classification of each potential metabolite by origin category (peptides, halogen-containing, MDM, drugs, nutrients, plants, natural products)
- Per-class tabulated counts (e.g., tagger_results.csv with 420 total: 125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, 148 natural products)
- Reduced dataset from RowMerger / REname for manual comparison among annotations

## How to apply

Download the working-example input table of putative annotations from the TurboPutative web server (https://proteomics.cnic.es/TurboPutative/) and submit it to the Tagger module, either through the web interface or programmatically via the RESTful API, to assign each potential metabolite an origin classification. Parse the Tagger output and tabulate counts of metabolites per origin class — peptides, halogen-containing, microbiota-dependent metabolites (MDM), drugs, nutrients, plants, natural products. As a validation case study, the atherosclerosis example yields 420 classified metabolites (125 peptides, 32 halogen-containing, 2 MDM, 59 drugs, 84 nutrients, 34 plants, 148 natural products); compare your tabulated counts against these reported totals when reproducing the example, and save the classification results as tagger_results.csv. For redundant or overlapping annotations, pair Tagger with REname (recognizing isomers/equivalent compounds and removing redundant data) and RowMerger (reducing dataset size for manual comparison), and use TPMetrics to score features by adduct probability and feature correlations when intensities and metadata are available.

## Related tools

- **TurboPutative** (Web platform whose Tagger module classifies putative annotations by origin; REname, RowMerger, and TPMetrics handle redundancy, dataset reduction, and adduct/correlation scoring) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Total number of classified metabolites matches the reported 420 for the atherosclerosis working example
- Per-class counts match the reported breakdown: 125 peptides, 32 halogen-containing, 2 microbiota-dependent metabolites, 59 drugs, 84 nutrients, 34 plants, 148 natural products
- Every metabolite in the input table receives exactly one origin classification in the Tagger output
- Results file (tagger_results.csv) is produced and its per-class counts are internally consistent with the summed total
- Redundant entries and isomers/equivalent compounds are removed when REname is applied alongside Tagger

## Limitations

- Tagger's classification 'provides preliminary insights into the biological system studied' — it is an initial triage of putative annotations, not validated metabolite confirmation
- Per-class counts are dataset-dependent: the working example detected only 2 microbiota-dependent metabolites, so rare classes may be near-zero in other datasets
- No discussion-section guidance or changelog is available from the source document for troubleshooting edge cases

## Evidence

- [abstract] TurboPutative web platform and its four modules: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [abstract] Tagger's role and preliminary nature: "Tagger classifies the different compounds and provides preliminary insights into the biological system studied"
- [abstract] RESTful API automation path: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] Entry-reduction rationale for the workflow: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
