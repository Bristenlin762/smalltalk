# SpeakEasy

**AI voice small-talk practice for English learners.**

SpeakEasy is a mobile app where learners practice conversations about current events or everyday scenarios, then review transcript-based coaching, vocabulary, cultural context, and conversation scores.

[Watch the demo on YouTube](https://youtu.be/BdcyFQ5QbK8) · [Original team repository](https://github.com/MaCanXH/smalltalk) · [Lulin He's fork](https://github.com/Bristenlin762/smalltalk)

## Demo

[![Watch the SpeakEasy demo on YouTube](https://img.youtube.com/vi/BdcyFQ5QbK8/hqdefault.jpg)](https://youtu.be/BdcyFQ5QbK8)

Click the thumbnail to watch the demo on YouTube.

## What the app does

- **Voice practice:** real-time conversations through the Vapi React Native SDK, with a default 3-minute session timer.
- **News-based topics:** RSS headlines and article context are transformed into beginner-friendly discussion topics using Groq.
- **Scenario practice:** preset and custom scenes specify a goal, partner role, setting, and personality.
- **Session feedback:** review user highlights, sentence upgrades, useful AI phrases, vocabulary, and cultural clues.
- **Progress and review:** revisit session results and save useful learning material.

## My Contributions

I am Lulin He, a contributor to this four-person Penn team project. My work covered the AI workflow, the initial news pipeline, conversation feedback, and the final demo video.

### AI Workflow and News Retrieval

- Proposed a dual-agent architecture separating real-time voice interaction from backend news retrieval and post-conversation evaluation.
- Developed the initial Google News RSS retrieval workflow and expanded its news coverage before the team's later multi-source feed expansion.
- Integrated Groq into the original Node.js backend to turn retrieved news into structured, conversation-ready context for the Vapi voice partner.
- Refined topic enrichment with related-news retrieval, article content, background details, quotations, timelines, vocabulary, and conversational framing.

### Evaluation and Learning Feedback

- Developed a hybrid feedback workflow combining six transcript- and timing-based scoring dimensions with Groq-generated coaching.
- Used conversation turns, vocabulary variety, approximate speaking rate, response delays, and pauses as inputs to local heuristic scoring.
- Added topic- and transcript-linked vocabulary explanations and cultural clues, alongside sentence upgrades, useful phrases, and follow-up suggestions.
- Preserved the existing scoring and feedback experience while integrating AI-generated learning content.

### Demo Video and Visual Storytelling

- Created the demo's narrative and complete production plan to explain the product and walk through the user experience.
- Wrote the hand-drawn storyboard and conceived the animation sequences used to communicate the app's concept.
- Produced and edited the final demo video, combining the visual narrative with the application walkthrough.

This work was developed with AI-assisted implementation tools. The project requirements, architecture proposal, iteration, integration, and demo direction formed my contribution to the team.

## Team Development

The original shared repository is maintained at [MaCanXH/smalltalk](https://github.com/MaCanXH/smalltalk). This repository is my fork of that team project.

The team subsequently expanded the news sources and caching pipeline and migrated the original Node.js backend to Supabase Edge Functions. The architecture and setup below describe the current shared implementation.

## Architecture

The voice and backend coaching workflows have separate responsibilities:

1. **Topic preparation:** the backend fetches RSS items, removes duplicate headlines, generates structured topic packs with Groq, and stores them in Supabase.
2. **Voice interaction:** the app requests per-call configuration from the backend and starts a Vapi voice session using the selected news or scene context.
3. **Post-session coaching:** the client calculates heuristic conversation scores and sends a speaker-labelled transcript and context to the Groq feedback endpoint.
4. **Results:** validated AI coaching is merged with the local result; local feedback remains available if the AI request fails.

### Current implementation details

These are configuration values and feature counts, not measured performance results.

| Component | Implementation |
| --- | --- |
| News ingestion | 8 configured RSS feeds, fetched concurrently with partial-failure handling |
| Headline pool | Up to 36 deduplicated news items before prompt-budget trimming |
| Topic generation | Requests a pack of 20 topics; actual output depends on available news and model response |
| Topic delivery | Returns 1–5 topics per request, defaulting to 3 |
| Cache | 60-minute freshness window, with background revalidation when supported and stale-cache fallback |
| Coaching | Groq-generated structured JSON with vocabulary, cultural clues, highlights, and conversational suggestions |
| Scores | 6 local heuristic dimensions: Vibe, Fluency, Tone, Confidence, Stamina, and Cultural Fit |

The scores are transcript- and timing-based practice indicators, not validated language assessments. The feedback pipeline does not evaluate raw audio, pronunciation, or accent.

## Technology

The original Groq integration used a Node.js backend; the current repository runs its backend on Supabase Edge Functions.

- **Mobile:** TypeScript, React Native, Expo SDK 54, Expo Router
- **Voice:** Vapi React Native SDK and Daily native dependencies
- **Backend:** Supabase Edge Functions using Deno and Groq
- **Storage and authentication:** Supabase and local AsyncStorage

## Code Guide

| Area | Source |
| --- | --- |
| RSS ingestion, topic generation, and cache | [`hotTopics.ts`](supabase/functions/api/hotTopics.ts) |
| Groq coaching endpoint | [`feedback.ts`](supabase/functions/api/feedback.ts) |
| Local heuristic scoring | [`scoring.ts`](lib/ai/scoring.ts) |
| Feedback validation and fallback | [`critic.ts`](lib/ai/critic.ts) |
| Vapi session configuration | [`vapiSession.ts`](supabase/functions/api/vapiSession.ts) |
| Voice session UI | [`active.tsx`](app/session/active.tsx) |
| Database schema | [`schema.sql`](supabase/schema.sql) |

## Development Setup

The app uses a native development build. It does not run in Expo Go because the Vapi voice stack requires native modules.

Apply [`supabase/schema.sql`](supabase/schema.sql) in your Supabase project before using database-backed features. [`supabase/scene_presets_seed.sql`](supabase/scene_presets_seed.sql) supplies the default scene presets. Configure a Vapi assistant for your own environment.

## Requirements

- Node and npm
- Xcode for iOS builds
- Android Studio for Android builds
- A Supabase project with the `api` and `vapi-webhook` Edge Functions deployed (`npx supabase functions deploy api vapi-webhook`) and the `GROQ_API_KEY`, `VAPI_ASSISTANT_ID`, `VAPI_PRIVATE_KEY`, `VAPI_ORG_ID`, and `VAPI_WEBHOOK_SECRET` secrets set (`npx supabase secrets set NAME=value`) — the app receives a short-lived Vapi token per call from the backend; no Vapi credential ships in env vars

## Environment

Create a local `.env` file:

```sh
EXPO_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=replace_with_supabase_anon_key
```

The backend URL is derived from the Supabase URL (`<project>/functions/v1/...`). To point the app at a non-default backend (e.g. a local `supabase functions serve`), set the optional `EXPO_PUBLIC_FEEDBACK_API_URL` override to that backend's `/api/feedback` URL.

## Install dependencies

Have to use --legacy-peer-deps because VAPI/Daily peer dependency conflict issue

```bash
npm install --legacy-peer-deps
```

## Local development

Build and install a local development client:

```bash
npx expo run:ios
# or
npx expo run:android
```

Start Metro for the dev client:

```bash
npx expo start --dev-client
```

If device discovery is unreliable, use:

```bash
npx expo start --dev-client --tunnel
```

Rebuild the native app after changing native dependencies or `app.json`.

## Physical iPhone testing

For Vapi voice testing, prefer a physical iPhone over the simulator.

Typical flow:

```bash
npx expo run:ios --device
npx expo start --dev-client
```

Requirements:

- iPhone connected to your Mac for the first install
- trusted computer pairing
- Developer Mode enabled on the phone if prompted
- Apple account configured in Xcode for signing

## EAS builds

This repo already includes EAS profiles in `eas.json`.

Development builds:

```bash
npx eas-cli@latest build --platform android --profile development
npx eas-cli@latest build --platform ios --profile development
npx eas-cli@latest build --platform ios --profile development-simulator
```

Preview build for sharing:

```bash
npx eas-cli@latest build --platform ios --profile preview
```

## Sharing with testers

For iOS, the `preview` profile uses internal distribution. That means the tester's iPhone must be registered for ad hoc provisioning before install.

Typical flow:

```bash
npx eas-cli@latest device:create
npx eas-cli@latest build --platform ios --profile preview
```

Then send the EAS build URL to the tester.

## Dependency note: Vapi and Daily peer mismatch

This project intentionally uses a newer Daily native stack than `@vapi-ai/react-native@0.3.0` declares in its peer dependencies:

- installed: `@daily-co/react-native-daily-js@0.86.0`
- installed: `@daily-co/react-native-webrtc@124.0.6-daily.1`
- Vapi peer metadata still points at the older `0.78.0` / `118.0.3-daily.4` line

This is deliberate. The older Daily stack pulls an older `daily-js` version that caused runtime compatibility problems in this app.

Practical impact:

- `npm install <package>` may fail with `ERESOLVE`
- local tool installs inside this repo can fail because npm re-checks the dependency graph

Recommended workflow:

```bash
npx eas-cli@latest <command>
npx expo install <package>
```

If you must install a non-Expo package and npm blocks on peer resolution:

```bash
npm install <package> --legacy-peer-deps
```

Avoid adding `eas-cli` as a local dependency in this repo.
