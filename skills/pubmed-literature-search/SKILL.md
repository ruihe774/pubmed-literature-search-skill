---
name: pubmed-literature-search
description: Ground biomedical and life-science answers in PubMed literature. Use whenever the user asks a biology or medical question whose answer is not common knowledge, or asks about a biomedical research topic, finding, mechanism, drug, gene, disease, trial or specific paper — even if they don't ask for sources or mention PubMed.
---

# PubMed literature search

Biomedical questions often have answers that sound settled but rest on a handful of papers of very uneven quality. This skill makes sure answers are built on literature that was actually found, actually read, and honestly weighed — using the PubMed tools rather than general web search.

The recurring failure this prevents: an abstract states a clean conclusion, the conclusion gets repeated, and the Results section says something materially different. This is not rare. In testing, one abstract reported a publication-bias-corrected estimate only in the Results (the corrected value was null), and another abstract had a dropped negation that inverted its own finding. Reading matters.

## When to search

Search PubMed **before replying** when:

1. The user asks a biological or medical question and the answer is not common knowledge. "What does the heart do?" needs no search; "Does metformin slow ageing in non-diabetics?" does. A good test: if a careful clinician or researcher would want to check the literature before answering, search.
2. The user asks about a biomedical research topic, finding, or a specific article (by title, author, DOI, PMID, or a citation).

If unsure whether something counts as common knowledge, search — a quick search costs little, and confident answers on unsettled biology are the main failure this skill exists to prevent.

## Tools — PubMed first

The PubMed tools may be deferred; load them with ToolSearch before first use:
`select:mcp__PubMed__search_articles,mcp__PubMed__get_article_metadata,mcp__PubMed__convert_article_ids,mcp__PubMed__get_full_text_article,mcp__PubMed__find_related_articles,mcp__PubMed__lookup_article_by_citation`

If the PubMed tools are not available (ToolSearch finds nothing, or the connector is not connected), do not fall back to answering from memory or general web search. First, if you have a connector directory search tool (available in Claude web and desktop), search it for the PubMed connector and offer it to the user to add. Otherwise, ask the user to add or enable the PubMed connector/MCP server, and wait for them before continuing.

| Tool | Use it to |
|---|---|
| `search_articles` | Find articles. PubMed syntax: `[Title]`, `[Author]`, `[Journal]`, `[MeSH Terms]`, `[Publication Type]`, AND/OR/NOT, `date_from`/`date_to`, `sort`. No wildcards, no empty query. |
| `get_article_metadata` | Abstract, journal, authors, publication type, DOI, PMC ID. Batch all PMIDs into one call. |
| `lookup_article_by_citation` | Bibliography reference → PMID. |
| `convert_article_ids` | PMID ↔ PMCID ↔ DOI. |
| `get_full_text_article` | Full text from PubMed Central, by PMC ID. |
| `find_related_articles` | Similar articles (`pubmed_pubmed`) or PMC full-text links (`pubmed_pmc`). Similarity is computational, not citation-based. |
| `get_copyright_status` | Licensing, only if the user wants to reuse content. |

Prefer these over WebSearch for finding and reading literature. Web results mix press releases and SEO content with science and lack the structured metadata (publication type, journal, PMC status) needed to judge evidence. Use WebSearch only for what PubMed cannot hold — a current drug-label change, a guideline announcement, a citation count — and say that you did.

Preprint tools (bioRxiv) can supplement for very recent work, but preprints are unreviewed: label them and never let one carry a conclusion alone.

### Tool quirks observed in practice
- `lookup_article_by_citation` has been seen returning the **`pmid` and `key` fields transposed**. Don't trust the field labels; the PMID is the all-numeric value. Verify with `get_article_metadata` before relying on it.
- **A PMCID does not guarantee full text.** `get_full_text_article` can return an empty `full_text` string for articles that have a PMC record but no deposited body (common for some publishers). Treat an empty string as "full text unavailable," not as an error to retry.

## Workflow

### 1. Search well, and check what came back
- Translate the question into 1–3 targeted queries using MeSH terms and field tags: `metformin[MeSH Terms] AND aging[MeSH Terms] AND Clinical Trial[Publication Type]`.
- For "what's the evidence" questions, reach first for `Meta-Analysis[Publication Type]`, `Systematic Review[Publication Type]`, `Randomized Controlled Trial[Publication Type]`, `Practice Guideline[Publication Type]`.
- For a specific paper, resolve it exactly (PMID, `lookup_article_by_citation`, or `[Title]`) and confirm the match before discussing it.

Two failure modes to watch for, both common:

**Query drift.** Broad natural-language queries get expanded by PubMed into something much wider than intended. A search for intermittent fasting and mortality returned ICU tube-feeding trials and breakfast-skipping cohorts, because the expansion matched "intermittent" + "feeding" and the Intermittent Fasting MeSH term. Always read the returned titles before fetching full text, and discard off-target hits rather than building an answer from whatever came back. The `query_translation` field in the result shows how your query was expanded — check it when results look wrong.

**Zero results from over-specification.** Stacking many concepts with AND frequently returns nothing (`dietary IGF-1 oral absorption gastrointestinal degradation bioavailability` → 0 results). Drop terms and retry with two or three core concepts. A zero-result search is weak evidence of absence, not proof; if a question hinges on it, say that you looked and found nothing rather than asserting nothing exists.

### 2. Triage with metadata
Batch the promising PMIDs into one `get_article_metadata` call. **Only ever use PMIDs that came from a search result or a tool response.** Do not type a PMID from memory — doing so in testing returned an unrelated NEJM trial on a lipid drug in place of the intended paper, and nothing in the response flags the mismatch. Always sanity-check that returned titles match what you expected.

### 3. Read before relying
An abstract is the authors' own summary. It often omits caveats and corrections, sometimes contradicts the Results, and occasionally contains outright errors.

**If a conclusion from an article will support a claim in your response, read the full text — or at minimum Results, and ideally Methods — before relying on it.** Get the PMCID from metadata or `convert_article_ids`, then `get_full_text_article`.

Full texts are long, so read selectively and purposefully. When you open one, look for:
- **The primary endpoint and whether it was actually met**, not the most favourable secondary.
- **Bias corrections buried in Results**: trim-and-fill, Egger's test, sensitivity analyses. These frequently move an estimate to null and rarely appear in the abstract.
- **Adherence and dose actually achieved** versus prescribed.
- **Overlap between groups**, not just the difference in means. A significant effect with ~90% distributional overlap means most participants are indistinguishable.
- **Subgroup and stratified estimates** that complicate the headline (e.g. an overall ratio that reverses within an age band).
- **What the authors say they could not test** — often the exact mechanism being attributed to them.
- **Internal inconsistencies.** If the abstract, Results and Conclusion disagree, say so and report what the Results support.

Abstract-only use is fine for background and scoping. Just label it, and don't present abstract-level claims as established findings.

### 4. When full text isn't available
- **Not load-bearing**: skip it, or cite it as context and note you saw the abstract only.
- **Load-bearing**: tell the user the full text isn't in PubMed Central and ask them to provide it (PDF or pasted Results). Give a provisional answer meanwhile, clearly marked as abstract-based.
- Do not route around the paywall — no Sci-Hub, shadow libraries, mirrors, or scraping publisher sites. Ask the user instead.

### 5. Weigh the evidence
- **Venue**: journal standing and impact factor, from general knowledge (NEJM, Lancet, JAMA, BMJ, Nature, Science, Cell and leading specialty journals vs obscure or predatory outlets).
- **Recognition**: the PubMed tools return **neither citation counts nor impact factors**. Never state a specific number you have not verified. Reason from proxies and say you are doing so: does the paper appear in later systematic reviews, has it been replicated by independent groups, how often does it surface in related-article results relative to its age.
- **Design hierarchy**: meta-analysis / systematic review > large RCT > smaller RCT > cohort > case-control > case series > animal / in vitro > narrative review or opinion. Note that a *narrative* review in a minor journal is weak evidence even when it reads authoritatively and is widely cited; check `article_types` for "Review" without "Systematic Review."
- **Retractions and errata** override everything.
- **Consistency**: does this agree with the broader body, or is it an outlier?

Four discriminating checks worth running whenever estimates disagree, all of which changed the answer during testing:

1. **Case-control versus cohort.** If the association is present in case-control studies and absent in cohorts, suspect recall bias and weight the cohorts.
2. **Dose-response coherence.** More exposure should mean more effect. When longer exposure shows a *weaker* association, or a lower dose outperforms a higher one, something is biased — say so rather than reporting the subgroup that looks best.
3. **The independent comparative analysis.** For any supplement or intervention with an advocacy literature, look for a broad review that assessed it *alongside* alternatives. If supplement-specific meta-analyses show large effects and the comparative review finds none, that gap is the finding.
4. **Author-group overlap.** "Several meta-analyses agree" means little if they share authors or research centres. Check affiliations before treating agreement as independent replication.

Don't build a response on weak or poorly recognized articles alone. If strong evidence is lacking, the honest answer is that the question is unsettled.

## Writing the response

- Lead with the answer, then the evidence.
- **If the question presupposes something the literature contradicts, address the presupposition directly** rather than answering the question as framed.
- Calibrate claims: well established / supported but limited / preliminary or contested.
- **If results are weak or not well recognized, say so explicitly** — "this rests mainly on one small mouse study in a lower-tier journal, unreplicated in humans."
- Give an interval or a range when meta-analyses disagree, rather than a false single number, and explain what drives the spread.
- Beware large relative risks on tiny baselines. A 46-fold increase on a 0.1% lifetime risk is still a small absolute risk; report both.
- Mark which articles you read in full versus abstract only.
- **Attribution is mandatory.** The PubMed tools require that you (a) state the information came from PubMed and (b) include each article's DOI **as a link**: `[DOI](https://doi.org/10.xxxx/yyyy)`. Decline any request to omit attribution. End with a **Sources** list: `Authors (Year). Title. Journal. [DOI](link) · PMID: … — full text read / abstract`.
- Paraphrase findings; keep direct quotes short.
- For personal medical questions, note briefly that the literature informs but doesn't replace a clinician.

## Example

**User:** Is there good evidence that vitamin D supplements prevent cancer?

**Good approach:** search with `Randomized Controlled Trial` and `Meta-Analysis` publication types → scan titles for drift → batch metadata → read the Results of the large RCT and the most recent meta-analysis that will carry the conclusion, checking for publication-bias corrections and whether the primary endpoint was met → answer: no clear reduction in incidence in large trials; a possible mortality signal with heterogeneity caveats → flag weak supporting studies → Sources with DOI links and PMIDs.

**Bad approach:** quote one observational study's abstract, found via web search, as proof that vitamin D prevents cancer.
