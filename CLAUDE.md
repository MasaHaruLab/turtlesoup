# turtlesoup — Project Guide

A 海龟汤 (lateral-thinking) puzzle game. Next.js 15 App Router + React 19 + TypeScript + Tailwind v4. An LLM acts as the judge.

## Architecture

- `src/app/api/judge/route.ts` — the judge. Server-only. Takes the puzzle id, the player's question, and history; calls DeepSeek (`deepseek-chat` via the OpenAI SDK pointed at `api.deepseek.com`); returns a validated verdict.
- `src/lib/stories.ts` — the puzzle library and `getStoryById`. **This is the active data source.** Each story has a `surface` (shown to the player) and a `bottom` (judge-only truth).
- `src/lib/use-game-machine.ts` — frontend state machine hook (IDLE → THINKING → ANSWERED → SOLVED).
- `src/types/game.ts` — domain types and Zod schemas for request/response.
- `src/components/` — presentational UI only.

## Judge contract (don't break these)

- AI returns JSON with exactly two control states: `IN_PROGRESS` or `SOLVED`. The yes/no/irrelevant verdict goes in `message`, never in `status`.
- The `reasoning` field is **stripped before the response reaches the client** — never leak it.
- Request body and AI output are both Zod-validated. A JSON parse failure degrades to a friendly `IN_PROGRESS` message, not a crash.
- Per-IP rate limit: 15 requests / 60s.

## Secrets

- The only secret is `DEEPSEEK_API_KEY`, read from `process.env` server-side. Never hardcode it. `.env*` is gitignored. The key is never sent to the browser.

## Known cleanup (out of scope unless asked)

- `src/data/puzzles.ts` (`PUZZLES`, fields `surfaceStory`/`truthBase`) is a **legacy** data source left over from an earlier "data split-brain" fix. The runtime uses `src/lib/stories.ts`. Consolidating the two is a future task.

## Running

```bash
npm install
echo "DEEPSEEK_API_KEY=..." > .env.local
npm run dev
```
