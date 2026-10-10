# pubmed-literature-search

A targeted (not "deep research") skill for literature research in PubMed that avoids "weak" papers. It answers biomedical questions from a handful of papers that were actually found, read and weighed, instead of repeating an abstract's headline.

> **Requires the PubMed connector.** The skill calls the PubMed tools (`search_articles`, `get_article_metadata`, `get_full_text_article`, and others). Without the connector it has nothing to search with. This plugin does not bundle or install it; enable it yourself (see [Requirements](#requirements)).

## What it does

When you ask a biology or medical question whose answer isn't common knowledge, or about a specific paper, Claude:

- Searches PubMed with targeted queries (MeSH terms, field tags, publication types) and checks the returned titles for query drift and off-target hits.
- Triages candidates with metadata, using only PMIDs that came from tool results.
- Reads the full text (from PubMed Central) before relying on a paper's conclusion, since abstracts often omit caveats, bias corrections or contradict the Results.
- Weighs the evidence by design hierarchy, venue, retractions, consistency and author overlap, and says so plainly when the evidence is weak or the question is unsettled.
- Answers with calibrated claims and a **Sources** list with DOI links and PMIDs, marking which papers were read in full and which only as abstracts.

It does not run a broad multi-source deep-research pipeline; it is meant for focused questions.

## Requirements

- Claude Code with plugin support.
- **The PubMed connector**, connected and enabled for your account (Claude.ai connectors, or the PubMed MCP server in your setup). The skill uses its `mcp__PubMed__*` tools.
- Optional: the bioRxiv connector for very recent preprints. The skill labels preprints as unreviewed and never relies on one alone.

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
