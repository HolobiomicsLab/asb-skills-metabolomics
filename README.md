<h1 align="center">ASB Skills: Metabolomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational metabolomics: mass spectrometry, NMR, annotation and data analysis.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-11%2C830-00284b" alt="11,830 skills">
  <img src="https://img.shields.io/badge/tools-1%2C989-00284b" alt="1,989 tools">
  <img src="https://img.shields.io/badge/workflows-20-00284b" alt="20 workflows">
  <img src="https://img.shields.io/badge/papers-477-00284b" alt="477 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the metabolomics collections of AgenticScienceBuilder (ASB). The current v3 collection contains 11,830 skills over 1,989 software tools and 20 composite workflows, distilled from 477 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load and that names its source paper. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline. The earlier v1 and v2 collections, and eight v2 technique packs, remain available.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/metabolomics/v1` | ASB Metabolomics Skill Collection v1 (legacy) | 1,485 | 292 | 2026-06-15 | not deposited |
| `collections/metabolomics/v2` | ASB Metabolomics Skill Collection v2 (expansion) | 5,859 | 909 | 2026-06-22 | [10.5281/zenodo.20794027](https://doi.org/10.5281/zenodo.20794027) |
| `collections/metabolomics/v3` | ASB Metabolomics Skill Collection v3 (consolidated rebuild, 20 composite workflows) | 11,830 | 1,989 | 2026-11-15 | [10.5281/zenodo.24255500](https://doi.org/10.5281/zenodo.24255500) |

## Status

Collection `v3`, built from corpus cut `0aff65e99ace`, is released on `main` at tag `metabolomics-v3.1.1` and archived on Zenodo under the version DOI [10.5281/zenodo.24255500](https://doi.org/10.5281/zenodo.24255500). The earlier `v1` and `v2` collections stay in the repository; they are not part of this release.

| Item | State |
|---|---|
| Release gate | [`receipts/0aff65e99ace/gate_report.json`](receipts/0aff65e99ace/gate_report.json) reports `warn`: 14 of 16 checks pass, 1 gives a warning and 1 is not applicable (`cut_manifest`). The warning concerns two open-access quote spans, of 418 and 386 characters, that exceed the 300-character snippet guidance. [`receipts/0aff65e99ace/verify.txt`](receipts/0aff65e99ace/verify.txt) records receipt and target re-verification with exit code 0. |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3` |
| DOI | [10.5281/zenodo.24255500](https://doi.org/10.5281/zenodo.24255500) (this version). The Zenodo deposit metadata is in [`collections/metabolomics/v3/.zenodo.json`](collections/metabolomics/v3/.zenodo.json). |
| Concept DOI | [10.5281/zenodo.20794026](https://doi.org/10.5281/zenodo.20794026) (all versions) |

## What is inside

| Item | Count | Path under `collections/metabolomics/v3/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 11,830 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 20 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 1,989 | `tools/` |
| Source papers | 477 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Consolidation map | 1 | `mapping.json` |
| Tool-licence records | 1 | `tool_licenses.json` |
| Offline search script | 1 | `bin/search_skills.py` |
| Optional semantic search script | 1 | `bin/semantic_search.py` |

The build and gate receipts are in `receipts/0aff65e99ace/` at the repository root. The tree follows the shared ASB layout, `collections/<domain>/<version>/` and `packs/<domain>/<technique>/`.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `compound-class-annotation` | untargeted LC-MS/MS features need class-level annotations from molecular formulas, CSI:FingerID fingerprints, CANOPUS and NPClassifier | 5 |
| `fbmn-annotation-propagation` | confident annotations must be propagated across feature-based molecular-network families | 6 |
| `gcms-deconvolution-and-identification` | GC-MS data need spectral deconvolution, EI library matching, retention-index validation and differential analysis | 5 |
| `gnn-spectral-property-prediction` | a graph neural network must predict a structure-dependent property to filter or re-rank candidates | 5 |
| `in-silico-biotransformation-prediction` | predicted biotransformation products of a parent structure must be screened against untargeted LC-MS/MS features | 5 |
| `ion-mobility-4d-annotation` | ion-mobility LC-IMS-MS/MS data need CCS-aware feature extraction, calibration, filtering and library matching | 4 |
| `lipidomics-lcms-annotation` | untargeted lipidomics data need preprocessing, lipid identification, rule validation, differential analysis and fusion | 6 |
| `masst-repository-scale-search` | a spectrum or feature must be searched across public metabolomics data with MASST and its specialized searches | 4 |
| `ms-imaging-spatial-metabolomics` | mass-spectrometry imaging data need spatial annotation, segmentation and region-wise comparison | 4 |
| `ms2lda-substructure-discovery` | recurring fragmentation motifs must be discovered with MS2LDA and mapped back to molecular-network features | 5 |
| `nmr-metabolomics-profiling` | NMR spectra or FIDs need preprocessing, metabolite identification, quantification and group statistics | 4 |
| `paired-omics-bgc-metabolite-linking` | paired genomic and metabolomic data must link biosynthetic gene clusters to metabolite features | 6 |
| `pathway-functional-analysis` | an LC-MS feature list needs mummichog functional analysis and pathway-level interpretation | 4 |
| `sirius-denovo-structure-elucidation` | unknown MS/MS features need formula, structure and class annotation without a spectral-library match | 5 |
| `spec2vec-ml-embedding-annotation` | MS2 spectra must be annotated by nearest-neighbour retrieval in a learned Spec2Vec embedding space | 4 |
| `stable-isotope-tracing-fluxomics` | stable-isotope tracing data need isotopologue distributions, natural-abundance correction and labelling statistics | 5 |
| `statistics-and-biomarker-discovery` | a metabolomics feature table needs normalization, statistical comparison, enrichment and biomarker selection | 5 |
| `suspect-screening-exposomics` | untargeted HRMS data must be screened against an environmental or exposure-related suspect list | 5 |
| `targeted-lcms-quantification` | targeted LC-MS measurements need integration, calibration, internal-standard normalization and batch QC | 5 |
| `untargeted-lcmsms-annotation` | untargeted LC-MS/MS data need preprocessing, networking, library and SIRIUS annotation, and a fused feature table | 6 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 7,433 |
| `noncommercial` | 1,651 |
| `restricted` | 276 |
| `unknown` | 2,470 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-metabolomics
claude plugin install metabolomics@asb-skills-metabolomics
```

`metabolomics` installs the latest release (v3). To pin v2 instead:

```bash
claude plugin install metabolomics-v2@asb-skills-metabolomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/metabolomics/v3/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-metabolomics/collections/metabolomics/v3/leaves/nmr-peak-picking <project>/.claude/skills/
```

The search script below prints the slug of each match. The [`USAGE.md`](collections/metabolomics/v3/USAGE.md) guide also documents selected-skill and EDAM-facet installs with `asbb`.

### v2 technique packs

The eight packs under [`packs/metabolomics/`](packs/metabolomics/README.md) are v2-only, overlapping views of v2 rather than subsets of v3. After registering this repository's marketplace, install only the packs you need, not the full collection as well:

| Plugin | Skills | Install |
|---|---:|---|
| `metabolomics-ce-ms` | 113 | `claude plugin install metabolomics-ce-ms@asb-skills-metabolomics` |
| `metabolomics-direct-infusion` | 97 | `claude plugin install metabolomics-direct-infusion@asb-skills-metabolomics` |
| `metabolomics-gc-ms` | 367 | `claude plugin install metabolomics-gc-ms@asb-skills-metabolomics` |
| `metabolomics-ion-mobility` | 385 | `claude plugin install metabolomics-ion-mobility@asb-skills-metabolomics` |
| `metabolomics-lc-ms` | 2,617 | `claude plugin install metabolomics-lc-ms@asb-skills-metabolomics` |
| `metabolomics-ms-generic` | 804 | `claude plugin install metabolomics-ms-generic@asb-skills-metabolomics` |
| `metabolomics-ms-imaging` | 291 | `claude plugin install metabolomics-ms-imaging@asb-skills-metabolomics` |
| `metabolomics-nmr` | 276 | `claude plugin install metabolomics-nmr@asb-skills-metabolomics` |

### Use

- Whole domain: install `metabolomics`; only the router is advertised and leaves load on demand.
- One task: `asbb install --skills a,b --collection metabolomics/v3 --repo <checkout> --runtime agents` (also `--skills-file` or `--query`).
- EDAM: `python3 collections/metabolomics/v3/bin/search_skills.py --edam topic_3172 -k 3` searches offline.
- End-to-end goal: add `--query "<goal>" --target workflows` to choose among 20 composite workflow outlines.
- Exact threshold: read the leaf's evidence and ground it against its source paper. The v3 checkout does not include the Perspicacité binder; the Zenodo archive adds it. The v2 collection and technique packs ship `bin/perspicacite_kb_bind.py`.
- Full routes, semantic cache and grounding: [`collections/metabolomics/v3/USAGE.md`](collections/metabolomics/v3/USAGE.md).
- Cite the collection's version DOI and the source DOI of every skill used.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-metabolomics
python3 collections/metabolomics/v3/bin/search_skills.py --query "spectral library matching" --tool GNPS -k 3
python3 collections/metabolomics/v3/bin/search_skills.py --target workflows --query "spectral library matching" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. spectral-library-matching--457b7be7e6bea314
2. spectral-library-matching--8e6198f5096cdf46
3. spectral-library-matching--a783b0b1b264a20a

1. gcms-deconvolution-and-identification
2. spec2vec-ml-embedding-annotation
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/0aff65e99ace/source.json` names the pipeline commit `2aa52f64`, the collections-tooling commit `68b83ea83` and the corpus commit `1fab24b`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

433 of the 477 included papers are open access according to `corpus.yaml` (`access.is_oa`). The other 44 have `is_oa: false` and are listed as `link-only`.

Each leaf names its source paper in `provenance.source_papers` (11,830 of 11,830 leaves). The gate parsed quote spans in 7,994 of them. The other 3,836 carry no evidence of either front-matter or body-span form (`gate_report.json`, `strip_verbatim_similarity` and `leaf_text_floor`).

This repository was split from `asb-skill-collections` at commit `f554641c2` (`main` after PR #58) on 2026-09-17 with path-filtered history, preserving file authorship and blobs. The receipt is under `receipts/split-2026-09-17/` (`source.json`, `digests.tsv`). Intent INT-ASB-042.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents and the community contribution loop.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- [`indicium-adapters-metabolomics`](https://github.com/HolobiomicsLab/indicium-adapters-metabolomics): the Indicium domain adapter for the same domain.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24255500](https://doi.org/10.5281/zenodo.24255500); its concept DOI, which covers all versions, is [10.5281/zenodo.20794026](https://doi.org/10.5281/zenodo.20794026).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.

## Contact

Louis-Félix Nothias, Holobiomics Lab (CNRS, Université Côte d'Azur): louis-felix.nothias@univ-cotedazur.fr.
Questions on a skill, a tool entry or a paper's use go to the issues of this repository first.
