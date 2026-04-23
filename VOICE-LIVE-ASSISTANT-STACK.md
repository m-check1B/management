# Voice Live Assistant Stack (2026-04-22)

## Decision
Use this as the default live voice stack now:
- STT: **Groq Whisper**
- Realtime LLM/voice: **xAI `grok-4-1-fast-non-reasoning`** via realtime API with `region=eu`
- Mode: **single-model smart lane** (no fast+smart duo by default)

## Why this is the current best fit
- Best practical speed/cost tradeoff in our tests.
- EU realtime routing is available on xAI path.
- Tool calling works in this lane.
- `grok-4.20-reasoning` is too slow for live callcenter UX.

## Language/understanding quality
- Keep **Groq STT** in front of Grok for better recognition of Matej's English in live calls.
- This is currently necessary for robust understanding quality.

## Tool behavior
- Tools are enabled and usable in live mode.
- Expect higher latency on tool-heavy turns vs no-tools turns.

## Latency expectation
- Not perfect, but acceptable for live assistant mode.
- Fast/no-tools turns are near real-time.
- Tool-heavy turns are slower but still workable.

## Operational note
- Default to this hybrid stack now.
- Escalate to heavier reasoning model only when needed for difficult turns.
