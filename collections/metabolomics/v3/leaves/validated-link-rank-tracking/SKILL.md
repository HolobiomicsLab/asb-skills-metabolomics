---
name: validated-link-rank-tracking
description: Use when reproducing or auditing NPClassScore-based prioritisation on
  PoDP paired omics datasets where known validated BGC–metabolite links (e.g. from
  MIBiG/PoDP records) must be checked for retention and rank improvement after filtering.
when_to_use_negative:
- Datasets whose validated metabolites lack MS/MS spectra entirely — no spectrum could
  be found for retimycin and nocardimicin, so no rank can be tracked for them
- 'Analyses that cannot compute structure-based classes: 1988 MS/MS spectra across
  the datasets had no CANOPUS or MolNetEnhancer prediction, and filtering behaviour
  differs substantially in that case'
- Linking tasks without prior co-occurrence candidate generation (e.g. no Metcalf-scored
  candidate list) — rank tracking is defined relative to the Metcalf-scored candidate
  ordering
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Platform used to load PoDP data, compute Metcalf-scored candidate links (get_links),
    and record validated-link ranks before and after filtering
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Class-matching score (cut-off 0.25) applied to filter candidate links, creating
    the post-filtering ranking to compare against
  repo: https://github.com/louwenjjr/mibig_classifications
- name: CANOPUS
  role: Provides structure-based compound classes (run via SIRIUS v4.9.3 on the GNPS
    mgf file) needed for NPClassScore scoring
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: cblaster
  role: Used with default settings to confirm BGC locations of validated links by
    querying MIBiG BGCs against the antiSMASH gbk files of each dataset
- name: BiG-SCAPE
  role: Groups BGCs into GCFs (docker run with cut-off 0.3) that form the units whose
    candidate-link counts and ranks are tracked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:16822e2cec7990056d73a91c236934721032343e52044d162b7f7d8edf42b3d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - CANOPUS
  - NPClassScore
  - NPLinker
  - antiSMASH
  - cblaster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# validated-link-rank-tracking

## Summary

Tracks the ranked positions of experimentally validated BGC–MS/MS spectrum links before and after NPClassScore filtering within the NPLinker platform, to verify that chemical class matching retains and prioritises known genome–metabolome links while reducing candidate lists.

## When to use

Use when reproducing or auditing NPClassScore-based prioritisation on PoDP paired omics datasets where known validated BGC–metabolite links (e.g. from MIBiG/PoDP records) must be checked for retention and rank improvement after filtering.

## When NOT to use

- Datasets whose validated metabolites lack MS/MS spectra entirely — no spectrum could be found for retimycin and nocardimicin, so no rank can be tracked for them
- Analyses that cannot compute structure-based classes: 1988 MS/MS spectra across the datasets had no CANOPUS or MolNetEnhancer prediction, and filtering behaviour differs substantially in that case
- Linking tasks without prior co-occurrence candidate generation (e.g. no Metcalf-scored candidate list) — rank tracking is defined relative to the Metcalf-scored candidate ordering

## Inputs

- PoDP paired omics datasets (MassIVE accessions MSV000078836, MSV000078839, MSV000079284, MSV000084950, MSV000084771)
- nplinker.toml configuration files (podp mode with dataset PoDP id)
- GNPS molecular networking output (mgf files)
- BiG-SCAPE GCF definitions (BiG-SCAPE cut-off 0.3) from antiSMASH 3/6 BGCs
- List of experimentally validated genome–metabolome links (PoDP/MIBiG records)
- Structure-based compound classes from CANOPUS (via SIRIUS v4.9.3) and MolNetEnhancer

## Outputs

- Ranked position of each validated link before NPClassScore filtering
- Ranked position of each validated link after NPClassScore filtering
- Candidate link counts per GCF before and after filtering (link-count tables)
- Per-dataset percentage reductions (68%/53%/69%) and average decrease (63%)

## How to apply

Instantiate NPLinker per PoDP dataset via a nplinker.toml config (podp mode with the dataset's PoDP id, e.g. MSV000078836/MSV000078839/MSV000079284 for Streptomyces/Salinispora, MSV000084950 for Cyanobacteria, MSV000084771 for Nocardia) and load data with load_data(). Compute candidate links with npl.get_links(npl.gcfs, "metcalf") using a standardised Metcalf cut-off of 2.5, lowered to 2 for the Nocardia dataset because its validated links had low Metcalf scores. Filter candidates at an NPClassScore cut-off of 0.25, discarding links lacking structure-based classes (CANOPUS/MolNetEnhancer predictions). For each validated link — located via cblaster against the antiSMASH gbk files where needed — record its ranked position before and after NPClassScore filtering, and confirm 96% of validated links are retained and prioritised while candidate links per GCF drop by 68% (Streptomyces/Salinispora), 53% (Cyanobacteria), and 69% (Nocardia), averaging 63%.

## Related tools

- **NPLinker** (Platform used to load PoDP data, compute Metcalf-scored candidate links (get_links), and record validated-link ranks before and after filtering) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Class-matching score (cut-off 0.25) applied to filter candidate links, creating the post-filtering ranking to compare against) — https://github.com/louwenjjr/mibig_classifications
- **CANOPUS** (Provides structure-based compound classes (run via SIRIUS v4.9.3 on the GNPS mgf file) needed for NPClassScore scoring) — https://github.com/kaibioinfo/canopus_treemap
- **cblaster** (Used with default settings to confirm BGC locations of validated links by querying MIBiG BGCs against the antiSMASH gbk files of each dataset)
- **BiG-SCAPE** (Groups BGCs into GCFs (docker run with cut-off 0.3) that form the units whose candidate-link counts and ranks are tracked)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- 96% of experimentally validated links across the three PoDP datasets are retained and prioritised after NPClassScore filtering
- Per-dataset reduction of candidate links per GCF matches the reported 68% (Streptomyces/Salinispora), 53% (Cyanobacteria, 27→13), and 69% (Nocardia, 206→64), averaging 63%
- Standardised Metcalf cut-off is 2.5 (lowered to 2 for the Nocardia dataset) and NPClassScore cut-off is 0.25; links below 0.25 or lacking structure-based classes are discarded
- Every validated link has a recorded ranked position both before and after NPClassScore filtering; ranks should improve or stay stable post-filtering for retained links
- cblaster queries of the PoDP-listed MIBiG BGC against the dataset antiSMASH gbk files confirm the BGC locations of the validated links

## Limitations

- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction; discarding such links is required to reach the reported reductions, while not filtering them yields only a 49% average decrease
- No MS/MS spectrum could be found for retimycin and nocardimicin, so those validated links cannot be rank-tracked
- The Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, requiring the threshold to be changed from 2.5 to 2 — method performance degrades on the smallest dataset
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are not well handled by current tools
- Merged antiSMASH 3 and antiSMASH 6 outputs were used across 50 strains, introducing version inconsistency; MolNetEnhancer must be run externally on the GNPS platform

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] 96% of validated links retained and prioritised: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] Record ranked positions of validated links before and after NPClassScore filtering: "The ranked position for each validated link was recorded before and after filtering with NPClassScore"
- [discussion] we used cblaster with default settings with the MIBiG BGC listed on the PoDP as query and the antiSMASH gbk files of each of the three datasets as database: "we used cblaster with default settings with the MIBiG BGC listed on the PoDP as query and the antiSMASH gbk files of each of the three datasets as database"
- [abstract] Average 63% reduction versus co-occurrence-only: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [methods] NPLinker load_data/get_links usage from methods docs: "# load data
npl.load_data()"
- [discussion] Nocardia Metcalf threshold change rationale: "the Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, which is why we changed the standardised Metcalf threshold"
