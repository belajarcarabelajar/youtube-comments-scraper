# YouTube Sentiment Analysis Skill Design

## Goal

Provide current, repo-scoped Codex guidance for maintaining and running the
YouTube comment sentiment pipeline without mixing it with YouTube API scraping
instructions.

## Skill Boundaries

### `youtube-comments-scraper`

Own YouTube Data API v3 concerns:

- API key safety and Bun usage
- `commentThreads.list` and `comments.list`
- pagination, reply collection, retries, and API errors
- mapping API comment snippets into `processComment`

Delegate sentiment implementation, evaluation, and taxonomy decisions to
`youtube-sentiment-analysis`.

### `youtube-sentiment-analysis`

Own the current local hybrid analysis pipeline:

- preprocessing Indonesian and English comment text
- spam and toxicity precedence
- transformer selection and lazy loading
- lexicon fallback for low-confidence results
- mixed sentiment classification
- optional Ollama verification in `processComment`
- output schema, benchmark evaluation, and regression testing

## Structure

Create:

```text
.agents/skills/youtube-sentiment-analysis/
├── SKILL.md
├── agents/openai.yaml
└── references/pipeline.md
```

Update:

```text
.agents/skills/youtube-comments-scraper/
├── SKILL.md
└── agents/openai.yaml
```

Keep `SKILL.md` concise. Put implementation-specific taxonomy and data-flow
details in `references/pipeline.md`, which the skill loads only for changes to
classification behavior, model selection, or output fields.

## Behavioral Rules

- Treat `src/index.ts`, `src/lexicons.ts`, `src/index.test.ts`, and
  `benchmark.json` as the current implementation evidence.
- Use Bun commands only.
- Keep inference local. Do not introduce cloud sentiment services.
- Preserve label precedence: `TOXIC`, then `SPAM`, then sentiment labels.
- Treat Ollama as optional verification with graceful fallback.
- Use tests and benchmark results before claiming accuracy improvements.
- Never expose `YOUTUBE_API_KEY` or raw secrets.

## Validation

- Run the skill creator's structural validator for both skills.
- Inspect generated `agents/openai.yaml` files.
- Run `bun test`.
- Review the final diff for stale lexicon-only instructions and overlapping
  skill descriptions.
