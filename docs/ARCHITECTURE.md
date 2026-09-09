# Feedprobe — System Architecture

Feedprobe is an agent-based simulation of social-media virality that runs entirely in the browser. A central **Agent/Controller** orchestrates personas, waves and actions; eight supporting components cover planning, tools, retrieval, external systems, memory/state, evaluation/verification, human interaction and failure handling. Everything lives in one self-contained file: `feedprobe.html`.

```
                     ┌─────────────────────────────┐
                     │          PLANNING           │
                     │  wave planner · seed setup  │
                     └──────────────┬──────────────┘
                                    │
  ┌───────────┐   ┌──────────────────────────────────┐   ┌───────────────────┐
  │   TOOLS   │──▶│        AGENT / CONTROLLER        │◀──│   MEMORY / STATE  │
  │ persona   │   │   simulate() — orchestrates      │   │  persona pool,    │
  │ generator │   │   personas, waves & actions      │   │  wave history,    │
  │ scoring   │   └────────────────┬─────────────────┘   │  action log       │
  └───────────┘                    │                     └───────────────────┘
                                   │
  ┌───────────┐   ┌────────────────┴─────────────────┐   ┌───────────────────┐
  │ RETRIEVAL │──▶│   EVALUATION / VERIFICATION      │◀──│      TOOLS        │
  │ keywords  │   │  computeReport() — metrics &     │   │  engagementProbs()│
  │ & tags    │   │  rule-based insights             │   │  propagation      │
  └───────────┘   └──────────────────────────────────┘   └───────────────────┘
```

## Component map

| Component | Responsibility | Where it lives in code |
|---|---|---|
| **Agent / controller** | Orchestrates the whole simulation: builds the pool, runs wave-by-wave propagation, routes every signal between components | `simulate(contentIn, audienceIn, cfg)` |
| **Planning** | Decides the seed audience size, wave count and outer-explore percentage from the UI config; picks interest clusters near the target | `pickClustersNear(targetTags, rng, count)`, config object in `simulate()` |
| **Tools** | The simulation toolkit: persona generator, content scoring, action model and the Explore propagation engine | `makePersona()`, `targetFitOf()`, `engagementProbs(p, v)`, propagation loop in `simulate()` |
| **Retrieval** | Ingests content — extracts keywords and tags from the uploaded file name and description, computes content–interest similarity | `extractKeywords(text, max)`, `tagSimilarity(a, b)` |
| **External systems** | The simulated platform surface: an Instagram-Explore-style feed that re-ranks and re-exposes the video; a plug-in point for a future vision/LLM brain (Whisper/CLIP) | Explore-style propagation logic in `simulate()`; ingestion boundary |
| **Memory / state** | Persists the persona pool and every decision: who was shown the video, who acted, and how each action propagated | persona pool + wave/action log carried through `simulate()` → `computeReport()` |
| **Evaluation / verification** | Turns wave history into the prediction report: reach, impressions, engagement rate, amplification, virality score, breakout % and rule-based insights | `computeReport(args)` |
| **Human interaction** | The UI: video upload, content attributes, target audience, simulation controls, Run / Replay-same-seed, and the live results dashboard | `runSim(seed)`, UI wiring in `feedprobe.html` |
| **Failure handling** | Determinism and honesty: a seeded RNG makes every run reproducible; zero-engagement runs report honestly instead of forcing results; malformed inputs degrade gracefully | `mulberry32(seed)` seeded RNG; guards in `simulate()` / `computeReport()` |

## Data flow

```
upload / describe video ──▶ retrieval (keywords, tags, duration)
        │
        ▼
planning ──▶ seed audience ──▶ agent/controller
        │                          │
        │   ┌──────────────────────┼──────────────────────┐
        │   ▼                      ▼                      ▼
        │ tools: persona pool   tools: content scoring   memory/state: log
        │   │                      │                      │
        │   └──────────▶ engagementProbs() ◀──────────────┘
        │                          │
        ▼                          ▼
evaluation/verification ◀── wave loop (Explore propagation, outer explore %)
        │
        ▼
report ──▶ human interaction (dashboard, insights, replay)
```

## Key properties

- **Deterministic per seed** — `mulberry32(seed)` drives every stochastic choice, so the same inputs and seed reproduce the exact same run. `Run` rolls a new seed; `Replay same seed` reproduces the previous result for A/B testing.
- **Self-contained** — zero dependencies; the whole system ships in one HTML file and runs offline.
- **Extensible** — the ingestion boundary is the plug-in point for a real video-understanding model, and engagement curves are tunable for calibration against platform data.