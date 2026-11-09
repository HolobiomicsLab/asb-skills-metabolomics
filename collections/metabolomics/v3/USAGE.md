# Using the ASB Metabolomics Collection (v3)

11,830 evidence-grounded skills, 1,989 software-tool records and 20 composite
workflows, distilled from 477 published method papers (journal articles and
preprints). This guide covers **choose a route → search → apply → ground → cite**.

## Which route for which need

| Need | Route |
|---|---|
| An agent runtime should know the whole domain | Install the full `metabolomics` plugin in Claude Code, or use `asbb install metabolomics --repo <checkout> --runtime agents` from the tooling package. |
| A handful of skills for one task | From an `asbb` installation plus this checkout, use `asbb install --skills`, `--skills-file` or `--query` (a fetch without a checkout is not available for v3 yet; see Install). |
| Skills for one EDAM topic or operation | Use `bin/search_skills.py --edam`, or filter `skills_index.json` directly; examples below. |
| An end-to-end analysis goal | Choose among the 20 composite workflows through `workflows/workflows_index.json`, `skills/_workflow_router/SKILL.md`, or `--target workflows`. |
| An exact parameter, threshold or default | Read the leaf's evidence and ground it against its source paper; do not infer precision from a generated summary. |

Not available in v3: technique packs (the eight packs in this repository are v2
content) and EDAM-based sub-collections (planned, not released). All v3 index
rows have an empty `techniques` list, so `--technique` cannot narrow this release.

## 0. Requirements

Browsing and searching need only the plain Markdown, JSON and YAML files.

| To… | You need |
|---|---|
| Install in Claude Code | Claude Code with plugin support |
| Run either collection search script | Python 3.10 or later; keyword mode uses only the standard library |
| Run semantic ranking | The dependencies above, NumPy, the optional embedding cache, and `OPENAI_API_KEY` to embed the query |
| Use `asbb` | Python 3.12 or later and the tooling checkout/package described under Install |
| Ground through Perspicacité | The Zenodo archive's standard-library binder and a running Perspicacité instance |
| Run a skill's scientific tool | That leaf's own dependencies; they are not bundled |

Read a leaf's frontmatter `tools` and `metadata.tools`, its procedure, and its
`tools_index.json` records before installing anything. The collection contains
instructions and metadata, not the scientific software itself.

## 1. Install

### Claude Code: the full collection

From a local checkout of this repository:

```bash
claude plugin marketplace add <checkout>
claude plugin install metabolomics@asb-skills-metabolomics
```

`metabolomics` resolves to v3; `metabolomics-v2` pins v2. Only
`skills/_router/SKILL.md` is advertised to the host. The 11,830 leaves remain
data under `leaves/` and are read on demand; do not enumerate them or load the
whole index into a prompt. Workflows have their own goal-level router.

### Other runtimes and selected skills: `asbb`

The `asbb` implementation and its documentation live in the separate
[asb-skill-collections tooling repository](https://github.com/HolobiomicsLab/asb-skill-collections).
Install that package from its checkout according to its README; it is not a
published package-index dependency of this collection. Selection installs need
both the tooling and this collection checkout; pass the latter as `--repo`.

From this repository root, the module form works without installing the package
when `PYTHONPATH` names the tooling checkout:

```bash
PYTHONPATH=<tooling-checkout> python -m asb_skill_collections.asbb_cli \
  install --skills nmr-peak-picking,gcms-spectral-deconvolution \
  --collection metabolomics/v3 --name my-annotation-set \
  --repo <checkout> --runtime agents --dry-run
```

```text
selection my-annotation-set: 2 skill(s) from metabolomics/v3
  nmr-peak-picking             open
  gcms-spectral-deconvolution  open
would install my-annotation-set (2 skill(s), 1 advertised entry(ies))
```

Drop `--dry-run` to install. `--skills-file <file>` reads one slug per line;
`--query "lipid class annotation" --top 3` selects the first three offline
search hits. `--dest <directory>` installs into an explicit directory instead
of a named runtime. A selection receives its own router and filtered indexes.

For the whole domain instead of a selection:

```bash
PYTHONPATH=<tooling-checkout> python -m asb_skill_collections.asbb_cli \
  install metabolomics --repo <checkout> --runtime agents --dry-run
```

```text
would write $HOME/.agents/skills/metabolomics-router
would install 1 skill(s) from metabolomics -> $HOME/.agents/skills
```

The one advertised entry is the router; the managed snapshot contains the full
corpus. Drop `--dry-run` to install it.

Without a checkout, `asbb fetch` is meant to retrieve named leaves and write a
provenance record. Its current implementation downloads from the tooling
repository `HolobiomicsLab/asb-skill-collections`, which no longer holds this
collection, so fetch is not available for v3 until the tooling learns the
domain repositories; use a checkout and a selection install instead.

## 2. Search

### Offline keyword, facet and EDAM search

From the repository root:

```bash
python collections/metabolomics/v3/bin/search_skills.py \
  --query "spectral library matching" --tool GNPS --json -k 3
```

The default target is `skills`; choose `--target workflows` or `--target tools`
for those indexes. `--tool` is an exact, case-insensitive tool-name filter.
`--max-tools N` applies the leaf registry/artifact guard (default 25; 0 disables
it). `--selector router|package` chooses the keyword rule, with
`ASB_SELECTOR_RULE` as the environment alternative. `--json` returns complete
ranked records, including score, matched fields, filters and qualified slug.
Without it, each hit prints a short description and the exact file to read.
No match exits 1; valid matches exit 0.

`--edam` matches a case-insensitive substring of either the operation IRI or any
topic IRI. With an empty query it browses the filtered set in deterministic
order; add words to rank within that set.

```bash
python collections/metabolomics/v3/bin/search_skills.py --edam topic_3520 -k 3
```

```text
1. 1d-cnn-training-on-metabolomic-signals
   Use when you have preprocessed SCMS metabolomics data ...
2. 2d-nmr-spectral-peak-filtering
   Use when preparing 2D NMR peak lists ...
3. ab-initio-molecular-dynamics-simulation
```

```bash
python collections/metabolomics/v3/bin/search_skills.py --edam operation_2945 -k 3
```

```text
1. ablation-study-benchmark-analysis
2. ablation-study-loss-and-decoder-comparison
3. adduct-abundance-quantification-per-dataset
```

Use full IRIs when integrating with EDAM-aware software. In this index,
`http://edamontology.org/topic_3172` is Metabolomics and
`operation_2945` is an analysis operation. A short Python filter is useful when the result
feeds another program:

```bash
python - <<'PY'
import json
from pathlib import Path
rows = json.loads(Path("collections/metabolomics/v3/skills_index.json").read_text())
matches = [r for r in rows if "http://edamontology.org/topic_3172" in r["edam_topics"]]
print(len(matches), "skills carry topic_3172")
for row in matches[:3]:
    print(row["slug"])
PY
```

```text
1625 skills carry topic_3172
1d-nmr-metabolomics-noise-thresholding
1d-nmr-peak-picking
absolute-lipid-concentration-quantification
```

Of the 11,830 rows, 10,788 have at least one EDAM topic and 8,773 have an EDAM
operation; 285 distinct operations occur. The most frequent topic IDs are
`topic_3520` (4,816 rows), `topic_0091` (2,233), `topic_3172` (1,625),
`topic_0121` (1,395) and `topic_0092` (893).

### `asbb search`

The tooling CLI reads this checkout offline:

```bash
PYTHONPATH=<tooling-checkout> python -m asb_skill_collections.asbb_cli \
  search "lipid class annotation" --collection metabolomics/v3 \
  --repo <checkout> --target skills -k 3
```

It returns the same JSON result dictionaries and supports `--technique`, but no
EDAM, tool or selector command-line filter. Use `bin/search_skills.py` for those.

### Optional semantic retrieval and the embedding cache

Nothing requires an embedding cache to install, browse, search or apply the
collection. Keyword retrieval is fully offline.

The Zenodo file `metabolomics-v3-leafemb.npz` contains one
`text-embedding-3-large` vector per leaf: `emb` is 11,830 × 3,072 float32 and
`slug` holds the corresponding 11,830 identifiers. The cache builder
L2-normalises the vectors, so dot product ranks cosine similarity. The cache
avoids re-embedding the corpus and provides the same leaf-embedding input used
when composing the workflows. It does not contain papers or executable models.

Put the file at `collections/metabolomics/v3/.cache/leafemb_v3.npz`, or point
`ASB_LEAF_EMB_CACHE` at it. Only the query is embedded at search time, so
`OPENAI_API_KEY` and network access are needed for semantic mode:

```bash
ASB_LEAF_EMB_CACHE=<path>/metabolomics-v3-leafemb.npz \
python collections/metabolomics/v3/bin/semantic_search.py \
  --query "lipid class annotation" --mode semantic -k 3
```

`--mode keyword` is explicitly offline. `--mode auto` tries semantic ranking,
then prints the reason on stderr and falls back to keyword ranking when the
cache, NumPy, key or query request is unavailable. Strict `--mode semantic`
never falls back: it exits 2 and prints the reason. The script defaults
`--collection` to its own v3 root and accepts `-k` like the keyword router.

## 3. Workflows: an end-to-end goal

A composite workflow is an ordered outline whose stages bind back to leaves of
this collection. Each `workflows/<slug>/SKILL.md` gives candidate leaves and
tools; its `workflow.yaml` declares typed inputs/outputs, `after` dependencies,
`inputs_from` links and grounding records. Follow the stages yourself: this
release has no automatic workflow runner, and stage structure was validated but
end-to-end execution was not.

Find a workflow with the router, the index, or:

```bash
python collections/metabolomics/v3/bin/search_skills.py \
  --query "untargeted LC-MS/MS annotation" --target workflows -k 3
```

```text
1. untargeted-lcmsms-annotation
2. compound-class-annotation
3. lipidomics-lcms-annotation
```

The 20 directory names in `workflows/` (excluding `_workflow_router`) are:

| Workflow | Stages |
|---|---:|
| `compound-class-annotation` | 5 |
| `fbmn-annotation-propagation` | 6 |
| `gcms-deconvolution-and-identification` | 5 |
| `gnn-spectral-property-prediction` | 5 |
| `in-silico-biotransformation-prediction` | 5 |
| `ion-mobility-4d-annotation` | 4 |
| `lipidomics-lcms-annotation` | 6 |
| `masst-repository-scale-search` | 4 |
| `ms-imaging-spatial-metabolomics` | 4 |
| `ms2lda-substructure-discovery` | 5 |
| `nmr-metabolomics-profiling` | 4 |
| `paired-omics-bgc-metabolite-linking` | 6 |
| `pathway-functional-analysis` | 4 |
| `sirius-denovo-structure-elucidation` | 5 |
| `spec2vec-ml-embedding-annotation` | 4 |
| `stable-isotope-tracing-fluxomics` | 5 |
| `statistics-and-biomarker-discovery` | 5 |
| `suspect-screening-exposomics` | 5 |
| `targeted-lcms-quantification` | 5 |
| `untargeted-lcmsms-annotation` | 6 |

The v2 composite `genome-scale-metabolic-flux-modeling` is not in v3: its
consolidating flux-report leaves were not realised in this rebuild. Atomic v3
leaves do cover genome-scale FBA/FVA and related operations, so this is a
missing composite route, not a blanket absence of flux-modelling skills.
`stable-isotope-tracing-fluxomics` is a different route: isotopologue and
labelling analysis, not genome-scale flux modelling.

## 4. Apply a leaf or a workflow

For one operation, read `leaves/<slug>/SKILL.md`. Check **When to use**, **When
NOT to use**, inputs, outputs, tool requirements, procedure and limitations.
The frontmatter carries EDAM terms, `provenance.source_papers` and
`metadata.license_tier`; the body carries short evidence quotations.

For a workflow, read its `SKILL.md` and `workflow.yaml`, then the candidate leaf
for each stage. A workflow is a composition aid, not evidence that the chained
programs have run together on your data. Preserve each stage's declared I/O
contract and validate intermediate outputs before continuing.

## 5. Ground against the source

Always start offline: read the leaf's evidence lines and the DOI/title records
in `provenance.source_papers`. The index row also has `dois`, and
`kb_bundle.json` maps every leaf to its source-paper KB identifiers. Obtain the
paper through its DOI and verify the exact parameter in context, including
supplementary material where needed.

The checkout currently ships `bin/search_skills.py` and
`bin/semantic_search.py`; it does **not** include the Perspicacité binder. The
Zenodo collection archive additionally bundles
`scripts/perspicacite_kb_bind.py` under the collection prefix. After extracting
that archive, from the collection directory:

```bash
python scripts/perspicacite_kb_bind.py resolve \
  --collection . --skill nmr-peak-picking
```

```text
{
  "skill": "nmr-peak-picking",
  "tier": "paper",
  "dois": ["10.1186/s13321-020-00481-0"],
  "kb_slugs": ["asb-paper-10-1186-s13321-020-00481-0"]
}
```

`resolve` is offline. With Perspicacité running at `PERSPICACITE_BASE`
(default `http://127.0.0.1:8000`), `prepare` creates and ingests the source KB;
`query` asks a cited question against it:

```bash
python scripts/perspicacite_kb_bind.py query --collection . \
  --skill <slug> --question "What exact threshold does the paper specify?"
```

`--tier paper` is the default; `--tier si` biases retrieval toward supplementary
material; `--tier repo` names source repositories instead. The binder's
`local` mode can clone repositories and optionally seek an open-access paper,
requiring Git, network and a destination (`--dest`). Treat generated answers as
pointers and read the cited source before adopting a threshold.

## 6. Licence tiers

Collection content is CC-BY-4.0; repository code is Apache-2.0. See
[`LICENSING.md`](../../LICENSING.md). A leaf's `license_tier` concerns the
underlying tool, not the CC-BY status of the skill text or the paper's reuse
rights. Read the tool's own licence before redistribution or commercial use.

| Tier | Leaves | Practical meaning |
|---|---:|---|
| `open` | 7,433 | Commercial use is permitted, subject to conditions such as copyleft. |
| `noncommercial` | 1,651 | Confirm an academic/noncommercial purpose; commercial use needs a separate licence. |
| `restricted` | 276 | A licence was established and constrains reuse; check it before commercial use or redistribution. |
| `unknown` | 2,470 | No tool-level licence evidence; verify before reliance and never vendor the tool as if licensed. |

The tier is in each index row and in the leaf's `metadata.license_tier`.

## 7. Metadata, attribution and citation

Useful files from the collection root:

| File | Contents |
|---|---|
| `skills_index.json` | One row per leaf: slug, name, description, EDAM fields, tools, DOIs, techniques, licence and triage metadata. |
| `tools_index.json` | Tool names, repositories, EDAM topics, licences and skill-use links. |
| `workflows/workflows_index.json` | Workflow descriptions, stages, member tools and binding metadata. |
| `kb_bundle.json` | Leaf-to-source DOI, tool and grounding-KB mapping. |
| `mapping.json` | Consolidation provenance: canonical members, capsules, aliases and source DOIs. |
| `corpus.yaml` | The 477 included source papers and their access records. |
| `collection.yaml`, `CITATION.cff`, `.zenodo.json` | Collection identity, roles, citation and deposit metadata. |

Cite both:

1. **The collection**, using v3's `CITATION.cff` and the version DOI of its
   Zenodo record, `10.5281/zenodo.24054446`; the lineage's concept DOI (all
   versions) is `10.5281/zenodo.20794026`.
2. **Every source paper behind a skill you used**, through the DOI in that
   leaf's `provenance.source_papers` block or its index row's `dois`.

`collection.yaml → roles` distinguishes the ASB generator, promoter and
sponsor. The authors in `CITATION.cff`/`.zenodo.json` are the collection
authorship record; they do not replace the papers' authors.

## 8. How it was generated and its provenance

ASB distilled method papers and, where available, public code repositories into
short, evidence-anchored procedures. V3 consolidated earlier class-A builds and
the November 2026 v3 wave into cut `0aff65e99ace`, deduplicated related task
outputs, and attached EDAM, tool-licence and triage metadata. The exact source
revisions and release receipts are under `receipts/0aff65e99ace/` in the
repository. `mapping.json` links consolidated leaves to their original members;
`corpus.yaml` records source access and licensing.

These are generated, source-grounded procedures, not independent validations of
every method. Triage labels and workflow outlines do not replace reading the
leaf, checking its licence and grounding critical details in the source paper.
