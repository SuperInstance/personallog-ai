# PersonalLog AI

**PersonalLog** is a privacy-first life-logging application that records, indexes, and retrieves personal events through natural-language search. It runs as a Cloudflare Worker, storing all data locally on the user's device — no cloud sync, no accounts, no ad tracking. Ask "When was the last time I went to the dentist?" and get an instant answer.

## Why It Matters

Human memory is lossy and unindexed. We forget dentist appointments, oil change mileage, dinner conversations, and medication schedules — then reconstruct them badly when needed. Existing solutions (notes apps, calendar) require manual organization; social platforms harvest personal data for advertising. PersonalLog occupies a different niche: capture everything effortlessly (voice, text, photo, location), auto-tag with NLP, and search with natural language. The privacy model is absolute: data never leaves the device. This makes it suitable for sensitive personal records — health, financial, relational — that users wouldn't trust to a cloud service.

## How It Works

### Capture Pipeline

```
Input (voice/text/photo) → NLP extraction → Auto-tagging → Local index → Search-ready
```

Each entry passes through:
1. **Transcription** (voice → text via Whisper)
2. **Entity extraction** (dates, people, places, quantities)
3. **Topic classification** (health, social, car, home, family, finance)
4. **Temporal indexing** (event timestamp + recurring reminder scheduling)

### Natural Language Search

The search engine parses natural queries into structured lookups:

```
"When did I last see Sarah?" →
  entity_filter = {people: ["Sarah"]}
  sort = descending(timestamp)
  limit = 1

"What was that restaurant Mike recommended?" →
  entity_filter = {people: ["Mike"], topics: ["food"]}
  keyword_filter = ["restaurant", "recommended"]
```

Complexity: O(k) for inverted-index lookup where k = postings list length. Sub-millisecond for personal corpora (<100K entries).

### Auto-Tagging Model

Tags are assigned via a lightweight on-device classifier:

```
P(tag | entry) = softmax(W · embedding(entry))
```

The embedding model runs in the browser via WebAssembly, producing 384-dimensional vectors. No server calls — the inference happens entirely on-device.

### Recurring Reminders

Temporal patterns trigger future reminders:

```
"Remind me in 6 months about the dentist" →
  reminder = {trigger: now + 6 months, query: "dentist"}

"Remind me at 50,000 miles about oil" →
  reminder = {trigger_condition: mileage ≥ 50000, query: "oil change"}
```

## Quick Start

```bash
# Deploy as Cloudflare Worker
npx wrangler deploy

# Local development
npx wrangler dev
```

The worker serves a responsive HTML dashboard at the root URL. All data persists in the browser's localStorage and IndexedDB.

## API

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Full dashboard with timeline, search, features |

*Client-side APIs: localStorage for entries, IndexedDB for embeddings, Web Speech API for voice input.*

## Architecture Notes

PersonalLog embodies η (eta) in the γ + η = C framework — it eliminates the *forgetting* that reduces personal competence C. By capturing everything and forgetting nothing, it removes the information-loss tax on daily life. The auto-tagging and search provide γ (constructive retrieval) — building answers from raw captured data. The privacy-first design ensures that η operates correctly: nothing leaks, nothing is exfiltrated. See [ARCHITECTURE.md](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md).

## References

1. Bush, V. (1945). "As We May Think." *The Atlantic*. — The original vision of personal memory augmentation (Memex).
2. Gemmell, J., et al. (2002). "MyLifeBits: Fulfilling the Memex Vision." *ACM Multimedia*. — Life-logging implementation.
3. Ebbinghaus, H. (1885). *Über das Gedächtnis*. — The forgetting curve that PersonalLog aims to flatten.

## License

MIT
