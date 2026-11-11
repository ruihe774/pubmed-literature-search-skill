# pubmed-literature-search

A skill for targeted (not "deep research") literature research in PubMed to avoid "weak" papers. It answers biomedical questions from a handful of papers that were actually found, read and weighed, instead of repeating an abstract's headline.

> **No connectors needed.** This version talks to NCBI E-utilities (and optionally the bioRxiv API) directly with `curl`. It only needs a shell with network access, `curl` and `jq`.

## What it does

When you ask a biology or medical question whose answer isn't common knowledge, or about a specific paper, Claude:

- Searches PubMed with targeted queries (MeSH terms, field tags, publication types) and checks the returned titles for query drift and off-target hits.
- Triages candidates with metadata, using only PMIDs that came from tool results.
- Reads the full text (from PubMed Central) before relying on a paper's conclusion, since abstracts often omit caveats, bias corrections or contradict the Results.
- Weighs the evidence by design hierarchy, venue, retractions, consistency and author overlap, and says so plainly when the evidence is weak or the question is unsettled.
- Answers with calibrated claims and a **Sources** list with DOI links and PMIDs, marking which papers were read in full and which only as abstracts.

It does not run a broad multi-source deep-research pipeline; it is meant for focused questions.

## Requirements

- Claude Code (or any agent) with plugin/skill support and a Bash tool.
- `curl` and `jq` on the machine (`xmllint` optional), with outbound HTTPS to `eutils.ncbi.nlm.nih.gov`, `www.ncbi.nlm.nih.gov`, `pmc.ncbi.nlm.nih.gov` and, for preprints, `api.biorxiv.org`.
- No API key required (NCBI allows ~3 requests/second without one).

## Data disclosure

This plugin contains no code of its own; it is instructions (a skill) that tell Claude which public endpoints to call with `curl`. **If the agent follows the skill's instructions, data from your conversation may be sent to NCBI and bioRxiv:**

- **PubMed / PMC (NCBI, U.S. National Library of Medicine).** Search queries, PMIDs, PMCIDs, DOIs and citation strings derived from your question are sent in request URLs. These may reflect what you asked, including any health or personal details you include.
- **bioRxiv / medRxiv (Cold Spring Harbor Laboratory).** Date ranges, categories and DOIs may be sent when looking for preprints.

The skill tells the agent not to include your name or email in requests. The plugin author does not receive, collect or store any of this data; the services' own terms and privacy policies apply. Do not include information in your questions that you are not comfortable sending to them.

## Intended audience

The skill is a research aid for adults and is not designed for or directed at children under 18. PubMed and bioRxiv index scientific and medical literature, which can include mature material such as clinical, sexual-health, reproductive, substance-use and mental-health content, graphic descriptions or images, and discussion of abuse or self-harm. These sources do not age-restrict their content, and the skill does not filter it.

## Installation

```
/plugin install pubmed-literature-search --marketplace ruihe774/pubmed-literature-search-skill
```

Or load a local checkout directly while trying it out:

```
claude --plugin-dir /path/to/pubmed-literature-search-skill
```

The skill (`pubmed-literature-search`) then triggers automatically on biomedical questions; you don't need to mention PubMed.

## Layout

- `.claude-plugin/plugin.json`: manifest
- `.claude-plugin/icon.png`: icon
- `.claude-plugin/marketplace.json`: single-plugin marketplace listing
- `skills/pubmed-literature-search/SKILL.md`: the skill

## Development

```
claude plugin validate .
```

## License

[The Unlicense](LICENSE)
