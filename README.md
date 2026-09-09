# 🔭 Feedprobe

**Agent-based social media virality prediction.** Upload your product demo and Feedprobe predicts how it will perform on an Instagram-style Explore feed *before you post* — by showing the video to a pool of AI personas who represent your target demographic (and people outside it), letting them act (watch / like / comment / share / skip), and propagating the video wave-by-wave to the personas most like the ones who engaged.

**Know how it performs before you post.**

- 🎬 Upload a product demo (or describe it) — metadata auto-extracts from the file
- 👥 200+ AI personas: demographics, interests and behaviour biases, biased toward your target audience plus a breakout pool
- 👍 Personas act on the content — driven by interest fit, hook strength and production quality
- 🔁 Explore-style propagation — engaged personas push the video to related personas, wave after wave
- 📈 Prediction: reach, impressions, engagement rate, amplification, virality score and breakout %
- 💡 Rule-based insights: hook strength, shareability, best-fit clusters and next steps

The simulation is **deterministic per seed** — same inputs + same seed ⇒ identical result, which makes every run replayable and A/B-testable.

---

## How it works

1. **Upload** — drop the video file; title, tags and duration auto-extract from the filename. Edit or override anything.
2. **Build personas** — a seeded pool of agents. ~60% are generated to match your target demographic (auto-detected from your audience description and interest chips); the rest represent the wider platform population *outside* your target.
3. **Agents act** — every persona who sees the video picks an action (`watch`, `like`, `comment`, `share`, `skip`) with probabilities from `engagementProbs()`, driven by content–interest fit plus the hook/quality you set.
4. **Explore propagation** — engaged personas push the video forward each wave: shares reach furthest, then comments, then likes, weighted by shared interest clusters, age and region. A small *outer explore %* mixes in off-target personas each wave to expose real breakout potential.
5. **Predict** — `computeReport()` turns the wave history into reach, impressions, engagement rate, amplification, virality score and breakout percentage.
6. **Insights** — rule-based findings explain *why* the prediction looks the way it does and what to change.

## Quick start

**Zero setup.** Open `feedprobe.html` in any modern browser (Chrome, Edge, Firefox, Safari). There is no server, no build step, no install — everything runs locally in the browser.

Or serve it anywhere static files work — e.g. deploy the single file to [Surge](https://surge.sh):

```bash
surge feedprobe.html feedprobe.surge.sh
```

## Dependencies

- **Runtime:** none. Pure client-side JavaScript (ES2020). The entire engine and UI ship in one self-contained HTML file.
- **Development (optional):** [Bun](https://bun.sh) — used only by the pitch-deck generators in `scratch/` (`bun run scratch/make_deck.js` → `feedprobe-deck.pptx`, `bun run scratch/make_pdf.js` → `feedprobe-deck.pdf`).

## Environment configuration

**None required.** There are no environment variables, secrets or keys. Every input — content metadata, target audience, pool size, seed, wave count, explore percentage — is configured directly in the UI.

## Repository structure

```
feedprobe/
├── feedprobe.html          # the entire application (simulation engine + UI)
├── docs/
│   └── ARCHITECTURE.md     # system architecture documentation
├── README.md
└── LICENSE
```

## Extending

- **Vision / LLM brain** — today the simulation is driven by metadata (title, description, tags, hook, quality). The ingestion layer is designed so a real video-understanding model (e.g. Whisper for transcripts, CLIP for frames) can plug in and replace the manual content attributes.
- **Calibration** — engagement curves can be tuned against historical post data so predictions track real platform behaviour.

## License

MIT — see [LICENSE](LICENSE).