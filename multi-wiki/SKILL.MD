---
name: multi-wiki
description: "Multi-industry Wiki — build and maintain isolated, interlinked markdown knowledge bases under ~/wiki/. Each industry (fashion, code, medical, etc.) is a fully independent sub-wiki with its own schema, preventing cross-contamination."
version: 3.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [wiki, knowledge-base, multi-domain, research, notes, markdown]
    category: research
    related_skills: [obsidian, arxiv, agentic-research-ideas]
---

# Multi-Industry Wiki

Build and maintain **isolated, industry-specific knowledge bases** under a single root directory `~/wiki/`.
Each industry sub-wiki is fully independent — its own schema, index, and log — preventing knowledge cross-contamination.

Based on [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), extended to multi-domain management.

**Core principle:** Knowledge never crosses sub-wiki boundaries. A medical concept stays in `medical/`. A fashion trend stays in `fashion/`. Code patterns stay in `code/`. The root layer only routes and coordinates — it never stores domain content.

**Division of labor:** The human curates sources and directs analysis. The agent summarizes, cross-references, files, and maintains consistency — always within the correct sub-wiki.

---

## When This Skill Activates

Use this skill when the user:
- Asks to create, build, or initialize a wiki or knowledge base (any industry)
- Asks to add a new industry/domain to the wiki system
- Asks to ingest, add, or process a source into a specific sub-wiki
- Asks a question that belongs to a specific domain
- Asks to lint, audit, or health-check one or all sub-wikis
- References "wiki", "knowledge base", or "notes" with an industry context

---

## Root Structure

**Root path:** `~/wiki/` (fixed; no env var override needed for multi-wiki mode)

```
~/wiki/
├── README.md               # Lists all registered sub-wikis and their domains
├── registry.md             # Machine-readable: sub-wiki name → path → domain → status
│
├── fashion/                # 服装 Wiki
│   ├── SCHEMA.md
│   ├── index.md
│   ├── log.md
│   ├── raw/
│   ├── entities/
│   ├── concepts/
│   ├── comparisons/
│   └── queries/
│
├── code/                   # Code Wiki
│   ├── SCHEMA.md
│   ├── index.md
│   ├── log.md
│   ├── raw/
│   ├── entities/
│   ├── concepts/
│   ├── comparisons/
│   └── queries/
│
├── medical/                # 医疗 Wiki
│   ├── SCHEMA.md
│   ├── index.md
│   ├── log.md
│   ├── raw/
│   ├── entities/
│   ├── concepts/
│   ├── comparisons/
│   └── queries/
│
└── [industry-name]/        # 新增行业按此模式扩展
    └── ...
```

> **Rule:** Content files ONLY live inside a sub-wiki directory. The root `~/wiki/` contains only `README.md`, `registry.md`, and sub-wiki folders — never domain content.

---

## Sub-Wiki Architecture (Three Layers)

Each sub-wiki (`~/wiki/<industry>/`) follows this structure:

```
<industry>/
├── SCHEMA.md           # Domain conventions, tag taxonomy, page thresholds
├── index.md            # Sectioned content catalog with one-line summaries
├── log.md              # Chronological action log (append-only, rotated yearly)
├── raw/                # Layer 1: Immutable source material
│   ├── articles/       # Web articles, clippings
│   ├── papers/         # PDFs, academic papers
│   ├── transcripts/    # Meeting notes, interviews
│   └── assets/         # Images, diagrams referenced by sources
├── entities/           # Layer 2: Entity pages (people, orgs, products, brands)
├── concepts/           # Layer 2: Concept/topic pages
├── comparisons/        # Layer 2: Side-by-side analyses
└── queries/            # Layer 2: Filed query results worth keeping
```

**Layer 1 — Raw Sources:** Immutable. Agent reads but never modifies.
**Layer 2 — The Wiki:** Agent-owned markdown files. Created, updated, cross-referenced by the agent.
**Layer 3 — The Schema:** `SCHEMA.md` defines structure, conventions, and tag taxonomy for this domain only.

---

## ISOLATION RULES (Critical — Never Violate)

These rules prevent knowledge cross-contamination across sub-wikis:

1. **No cross-wiki wikilinks.** `[[wikilinks]]` only point to pages within the same sub-wiki. Never write `[[../medical/drug-x]]` from inside `fashion/`.
2. **No shared tag taxonomies.** Each sub-wiki's `SCHEMA.md` defines its own tags. A tag like `brand` in `fashion/` has no relation to a `brand` tag in `code/`.
3. **No cross-wiki entity pages.** If a concept appears in two industries (e.g., "AI" in both `code/` and `medical/`), create **separate pages** in each sub-wiki with domain-specific framing. Do not link them.
4. **Source files stay in their sub-wiki's `raw/`.** Never place a medical paper in `code/raw/`.
5. **Ambiguous sources go to the most specific sub-wiki.** If unsure, ask the user before ingesting.
6. **Log entries are sub-wiki-local.** Each sub-wiki has its own `log.md`. There is no root-level log.

> **Why:** Cross-referencing medical and fashion knowledge produces noise, not insight. Isolation ensures each sub-wiki grows as a coherent, domain-expert knowledge base.

---

## Session Start Protocol

### Step 1: Identify the target sub-wiki

Before doing anything, determine which sub-wiki is relevant:

```bash
# Read the registry to know what sub-wikis exist
cat ~/wiki/registry.md
```

Ask the user if the domain is unclear: *"This looks like it could belong to `medical/` or `code/` — which sub-wiki should I use?"*

### Step 2: Orient within the target sub-wiki

Once the sub-wiki is identified, **always orient before acting**:

```bash
SUBWIKI=~/wiki/<industry>

# ① Read schema — conventions and tag taxonomy
cat "$SUBWIKI/SCHEMA.md"

# ② Read index — what pages already exist
cat "$SUBWIKI/index.md"

# ③ Read recent log — what happened recently
tail -50 "$SUBWIKI/log.md"
```

Only after orientation: ingest, query, or lint. This prevents duplicates, missed cross-references, and schema violations.

For large sub-wikis (100+ pages), also search before creating:

```bash
grep -r "keyword" "$SUBWIKI" --include="*.md" -l
```

---

## Operations

### 1. Initialize a New Sub-Wiki

When the user asks to add a new industry to the wiki system:

```bash
INDUSTRY="<industry-name>"   # e.g. fashion, code, medical, finance, legal
SUBWIKI=~/wiki/$INDUSTRY
mkdir -p "$SUBWIKI"/{raw/{articles,papers,transcripts,assets},entities,concepts,comparisons,queries}
```

Then:

1. Ask the user: *"What does this sub-wiki cover? Give me 1-2 sentences describing the domain scope."*
2. Write `SCHEMA.md` customized to the domain (see template below)
3. Write initial `index.md`
4. Write initial `log.md` with creation entry
5. **Register the sub-wiki** — append to `~/wiki/registry.md`
6. **Update `~/wiki/README.md`** — add to the sub-wiki list
7. Confirm and suggest first sources to ingest

#### SCHEMA.md Template (customize per industry)

```markdown
# <Industry> Wiki Schema

## Domain
[Precise scope — what this sub-wiki covers and what it explicitly excludes]

Example scopes:
- fashion/: Garment industry, brands, trends, materials, supply chain, retail. Excludes general business strategy.
- code/: Software engineering, languages, frameworks, tools, algorithms, architecture. Excludes non-technical project management.
- medical/: Diseases, treatments, drugs, clinical research, anatomy, public health. Excludes wellness lifestyle content.

## Isolation Note
This wiki is isolated. Do NOT link to pages in other sub-wikis.
Do NOT import tag taxonomies from other sub-wikis.
If a concept overlaps with another domain, maintain a separate page here with domain-specific framing.

## Conventions
- File names: lowercase, hyphens, no spaces (e.g., `transformer-architecture.md`)
- Every wiki page starts with YAML frontmatter (see below)
- Use `[[wikilinks]]` to link between pages within this sub-wiki only
- When updating a page, always bump the `updated` date
- Every new page must be added to `index.md` under the correct section
- Every action must be appended to `log.md`

## Frontmatter
  ```yaml
  ---
  title: Page Title
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  type: entity | concept | comparison | query | summary
  tags: [from taxonomy below]
  sources: [raw/articles/source-name.md]
  wiki: <industry>
  ---
  ```
  Note: The `wiki: <industry>` field identifies which sub-wiki this page belongs to.

## Tag Taxonomy
[Define 10-20 top-level tags specific to this domain. Add new tags here BEFORE using them.]

### Example — fashion/:
- Products: garment, accessory, footwear, material, fabric
- Industry: brand, designer, retailer, manufacturer, supplier
- Trends: season, collection, style, colorway, silhouette
- Business: pricing, distribution, sustainability, collaboration
- Meta: comparison, timeline, controversy, prediction

### Example — code/:
- Tech: language, framework, library, tool, algorithm, architecture
- Concepts: pattern, paradigm, protocol, standard, specification
- People/Orgs: person, company, open-source, community
- Practice: performance, security, testing, deployment, scalability
- Meta: comparison, benchmark, controversy, tutorial

### Example — medical/:
- Clinical: disease, syndrome, symptom, diagnosis, treatment, drug
- Science: anatomy, physiology, mechanism, pathway, biomarker
- Research: trial, study, meta-analysis, guideline, evidence
- Specialties: cardiology, oncology, neurology, immunology, etc.
- Meta: comparison, controversy, case-study, protocol

## Page Thresholds
- **Create a page** when an entity/concept appears in 2+ sources OR is central to one source
- **Add to existing page** when a source mentions something already covered
- **DON'T create a page** for passing mentions, minor details, or things outside the domain scope
- **Split a page** when it exceeds ~200 lines — break into sub-topics with cross-links
- **Archive a page** when its content is fully superseded — move to `_archive/`, remove from index

## Update Policy
When new information conflicts with existing content:
1. Check dates — newer sources generally supersede older ones
2. If genuinely contradictory, note both positions with dates and sources
3. Mark contradiction in frontmatter: `contradictions: [page-name]`
4. Flag for user review in lint report
```

#### registry.md Template

The root registry tracks all sub-wikis:

```markdown
# Wiki Registry

| Sub-wiki   | Path              | Domain                        | Created    | Status |
|------------|-------------------|-------------------------------|------------|--------|
| fashion    | ~/wiki/fashion/   | Garment industry & trends     | YYYY-MM-DD | active |
| code       | ~/wiki/code/      | Software engineering & tools  | YYYY-MM-DD | active |
| medical    | ~/wiki/medical/   | Medical research & clinical   | YYYY-MM-DD | active |
```

#### README.md Template

```markdown
# Wiki Root

Multi-industry knowledge base. Each sub-directory is an isolated wiki domain.

## Sub-Wikis

- **fashion/** — 服装行业：品牌、设计师、面料、趋势、供应链
- **code/** — 软件工程：语言、框架、工具、算法、架构
- **medical/** — 医疗健康：疾病、治疗、药物、临床研究

## Rules

- Content only lives inside sub-wikis. This root contains only routing files.
- Sub-wikis are isolated — no cross-wiki links or shared taxonomies.
- To add a new industry: initialize a sub-wiki and update registry.md + README.md.
```

---

### 2. Ingest a Source

When the user provides a source to add:

① **Identify the sub-wiki** — confirm with the user if ambiguous. A source about "medical-grade textiles" might belong in `medical/` OR `fashion/` — ask.

② **Orient within the target sub-wiki** (SCHEMA → index → log tail)

③ **File the raw source:**
```bash
cp <source> ~/wiki/<industry>/raw/articles/<slug>.md
# or for PDFs:
cp <source>.pdf ~/wiki/<industry>/raw/papers/<slug>.pdf
```

④ **Discuss takeaways** with the user — what's interesting for this domain? (Skip in automated contexts.)

⑤ **Check for existing pages** — search index.md and grep for mentioned entities/concepts within this sub-wiki only.

⑥ **Write or update wiki pages:**
   - **New entities/concepts:** Create only if they meet Page Thresholds in this sub-wiki's SCHEMA.md
   - **Existing pages:** Add new info, update facts, bump `updated` date. Follow Update Policy on conflicts.
   - **Cross-reference:** Every new/updated page must link to ≥2 other pages within the same sub-wiki via `[[wikilinks]]`.
   - **Tags:** Only use tags from this sub-wiki's taxonomy in SCHEMA.md. Never import tags from other sub-wikis.

⑦ **Update navigation:**
   - Add new pages to `index.md` under correct section, alphabetically
   - Update "Total pages" count and "Last updated" date in index header
   - Append to `log.md`: `## [YYYY-MM-DD] ingest | Source Title`
   - List every file created or updated in the log entry

⑧ **Report what changed** — list every file created or updated to the user.

A single source can trigger updates across 5-15 wiki pages. This is normal — it's the compounding effect.

---

### 3. Query

When the user asks a question:

① **Identify the sub-wiki** — which domain does this question belong to? If it spans multiple domains (e.g., "How is AI used in medical diagnosis?"), answer from each relevant sub-wiki separately, clearly labeled.

② **Read `index.md`** of the target sub-wiki to identify relevant pages.

③ **For sub-wikis with 100+ pages**, grep for key terms across all `.md` files.

④ **Read the relevant pages** within the sub-wiki.

⑤ **Synthesize an answer** citing wiki pages: "Based on [[page-a]] and [[page-b]] in the `medical/` wiki..."

⑥ **File valuable answers back** — if the answer is a substantial comparison or novel synthesis, create a page in `queries/` or `comparisons/` within the same sub-wiki. Don't file trivial lookups.

⑦ **Update `log.md`** of the sub-wiki with the query and whether it was filed.

> **Cross-domain queries:** When a question genuinely spans two domains, answer from each sub-wiki in separate sections. Make it clear which sub-wiki each insight comes from. Do not merge the knowledge.

---

### 4. Lint a Sub-Wiki

When the user asks to audit a sub-wiki (or all of them):

```bash
# Lint a specific sub-wiki
SUBWIKI=~/wiki/<industry>

# Or lint all sub-wikis by iterating registry.md
```

Lint checks (run within each sub-wiki):

① **Orphan pages:** Pages with no inbound `[[wikilinks]]` from other pages in the same sub-wiki.

② **Broken wikilinks:** `[[links]]` pointing to pages that don't exist within the sub-wiki.

③ **Cross-wiki link violations:** Any `[[link]]` or path reference pointing outside the sub-wiki directory. These are schema violations — flag and remove.

④ **Index completeness:** Every wiki page should appear in `index.md`. Compare filesystem against index entries.

⑤ **Frontmatter validation:** Every page must have all required fields (title, created, updated, type, tags, sources, wiki). Tags must be in this sub-wiki's taxonomy. `wiki:` field must match the sub-wiki name.

⑥ **Stale content:** Pages whose `updated` date is >90 days older than the most recent source mentioning the same entities.

⑦ **Contradictions:** Pages sharing tags/entities but stating conflicting facts.

⑧ **Page size:** Flag pages over 200 lines — candidates for splitting.

⑨ **Tag audit:** List all tags in use, flag any not in SCHEMA.md taxonomy.

⑩ **Log rotation:** If `log.md` exceeds 500 entries, rotate it to `log-YYYY.md`.

⑪ **Report findings** with specific file paths and suggested actions, grouped by severity:
   `cross-wiki violations > broken links > orphans > stale content > style issues`

⑫ **Append to sub-wiki log.md:** `## [YYYY-MM-DD] lint | N issues found`

---

## Working With the Wiki

### Search Within a Sub-Wiki

```bash
SUBWIKI=~/wiki/<industry>

# Find pages by content
grep -r "keyword" "$SUBWIKI" --include="*.md" -l

# Find pages by tag
grep -r "tags:.*<tag>" "$SUBWIKI" --include="*.md" -l

# Recent activity
tail -30 "$SUBWIKI/log.md"
```

> Never search across sub-wikis for content queries. Always scope to the relevant sub-wiki.

### Bulk Ingest

When ingesting multiple sources at once into a single sub-wiki:
1. Confirm all sources belong to the same sub-wiki (ask if unsure)
2. Read all sources first
3. Identify all entities and concepts across all sources
4. Check existing pages for all of them (one search pass)
5. Create/update pages in one pass
6. Update `index.md` once at the end
7. Write a single log entry covering the batch

### Adding a New Industry

When the user wants to add a new domain (e.g., `finance/`, `legal/`, `cooking/`):
1. Confirm the domain scope and name with the user
2. Initialize the sub-wiki (see "Initialize a New Sub-Wiki")
3. Update `~/wiki/registry.md` — add a row
4. Update `~/wiki/README.md` — add to the sub-wiki list
5. Write a domain-appropriate `SCHEMA.md` with a precise scope definition that excludes overlap with existing sub-wikis

### Archiving

When content is fully superseded or scope changes:
1. Create `_archive/` in the sub-wiki if it doesn't exist
2. Move the page to `_archive/` preserving the original path
3. Remove from `index.md`
4. Update linking pages — replace wikilink with plain text + "(archived)"
5. Log the archive action in the sub-wiki's `log.md`

### Obsidian Integration

Each sub-wiki directory works as a standalone Obsidian vault:
- Open `~/wiki/fashion/` as one vault, `~/wiki/medical/` as another
- Do NOT open `~/wiki/` itself as a vault — this would allow accidental cross-wiki links
- `[[wikilinks]]` render as clickable links within the vault
- YAML frontmatter (including `wiki:` field) powers Dataview queries
- The `raw/assets/` folder holds images referenced via `![[image.png]]`

Recommended Obsidian setup per sub-wiki vault:
- Set attachment folder to `raw/assets/`
- Enable "Wikilinks" in settings
- Install Dataview plugin for domain-specific queries:
  ```
  TABLE tags, updated FROM "concepts" WHERE contains(tags, "treatment")
  ```

---

## Pitfalls

- **Never cross-wiki link.** `[[wikilinks]]` are sub-wiki-local. Cross-wiki knowledge sharing creates noise, not insight.
- **Never place a source in the wrong sub-wiki's `raw/`.** A medical paper in `code/raw/` will corrupt that sub-wiki's knowledge.
- **Always orient first.** Read SCHEMA + index + log tail before any operation. Skipping causes duplicates and missed cross-references.
- **Always update `index.md` and `log.md`.** These are the navigational backbone. Skipping degrades the wiki.
- **Confirm sub-wiki before acting on ambiguous sources.** A source about "surgical robotics" could be `medical/` or `code/`. Ask.
- **Tags are local.** Never reuse a tag definition from another sub-wiki. Add it to this sub-wiki's SCHEMA.md independently.
- **Don't create pages for passing mentions.** Follow Page Thresholds in SCHEMA.md.
- **Frontmatter is required**, including the `wiki:` field. It enables per-sub-wiki search and cross-contamination audits.
- **Keep pages scannable.** Split pages over 200 lines. Every page should be readable in 30 seconds.
- **Ask before mass-updating.** If an ingest would touch 10+ existing pages, confirm scope first.
- **Rotate logs.** When `log.md` exceeds 500 entries, rename to `log-YYYY.md` and start fresh.
- **Handle contradictions explicitly.** Don't silently overwrite. Note both claims with dates, mark in frontmatter, flag for review.
- **Never open `~/wiki/` root as an Obsidian vault.** Always open individual sub-wiki directories.
