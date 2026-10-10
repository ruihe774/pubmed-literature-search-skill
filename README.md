# pubmed-literature-search

A skill for targeted (not "deep research") literature research in PubMed to avoid "weak" papers. It answers biomedical questions from a handful of papers that were actually found, read and weighed, instead of repeating an abstract's headline.

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

## Data disclosure

This plugin contains no code and makes no network requests itself. It is instructions (a skill) that tell Claude how to use the PubMed connector and, optionally, the bioRxiv connector. **If the agent follows the skill's instructions, data from your conversation may be sent to PubMed or bioRxiv:**

- **PubMed (NCBI / U.S. National Library of Medicine).** The agent derives search queries, PMIDs, DOIs and citation strings from your question and sends them to PubMed through the connector's tools (`search_articles`, `get_article_metadata`, `get_full_text_article`, and others). Those terms may reflect what you asked, including any health, medical or personal details you put in your question.
- **bioRxiv / medRxiv (Cold Spring Harbor Laboratory).** If the optional bioRxiv connector is enabled, the agent may send search parameters (date ranges, subject categories, DOIs) to bioRxiv to look for recent preprints.

Requests go through the connector you have enabled, so the connector provider's own terms and privacy policy also apply, as do those of PubMed and bioRxiv. The plugin author does not receive, collect or store any of this data. Do not include information in your questions that you are not comfortable sending to these services. Disable the connectors to prevent any such transmission.

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
