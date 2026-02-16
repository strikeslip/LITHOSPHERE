# SOS Development Plan
### Sounds of Seismic — Electronica Earthwork
**Prepared:** Mon 16 Feb 2026  
**Source Analysis:** "The Lithospheric Conductor: Architecting an Agentic Earthwork" (Google AI Studio thread, 16 Feb 2026)  
**Live Reference:** [sos.allshookup.org](https://sos.allshookup.org) · [allshookup.org/seismic-field.html](https://allshookup.org/seismic-field.html)

---

## 0. Purpose

The Lithospheric Conductor thread walks through a conceptual architecture for upgrading SOS from its current pure-JavaScript, dependency-free browser implementation to a TypeScript-driven system with explicit state management, off-thread audio processing, and optional LLM commentary.

This plan separates what's real from what's aspirational, identifies what the thread gets right, flags where it drifts into generated slop, and lays out a phased execution path that respects the existing SOS codebase, SHOOK's artistic intent, and the practical constraints of a solo interdisciplinary practice.

Total timeline: **6 months** (26 weeks). Four phases. Each phase ships something playable.

---

## 1. Current State

### Production Instruments

| Interface | URL | Function |
|---|---|---|
| **SOS Home** | sos.allshookup.org | Landing, tectonic data loader, navigation |
| **ONE** | /ONE.html | Seismic ambient electronica |
| **SEISFLOW** | /flow.html | Melodic mapping of seismic wave frequencies |
| **SEISTRONICA** | /seis.html | Classic electronica + seismic data |
| **ShadowZone** | /ShadowZone.html | Three-layer granular synthesis, scale/root/stretch controls |
| **Kamchatka M8.8** | /synths/Kamchatka-8-8-Synth.html | Event-specific sonification (2025 Kamchatka 8.8) |
| **ANMO FM Synth** | /synths/ANMO-FM-Synth.html | Station-specific AI synthesizer |
| **Seismic Field** | allshookup.org/seismic-field.html | M6+ global earthquake archive since 1900 |

### Technical Stack

- Pure JavaScript, zero external dependencies
- Web Audio API (native browser)
- USGS GeoJSON feeds + EarthScope/IRIS MiniSEED (BHZ)
- Custom Earthquake Sound Engine (ESE)
- Algorithmic granular synthesis
- Static hosting, GitHub at [github.com/strikeslip](https://github.com/strikeslip)

### What Works

The dependency-free philosophy is a genuine strength. Every synth page loads instantly, runs anywhere, has zero supply-chain risk. The conceptual framework (Smithson's site/non-site, planetary non-site, seismic unconscious) is rigorous. The instrument collection shows real artistic range.

---

## 2. Lithospheric Conductor — Critical Reading

### What the Thread Gets Right

**Type safety for seismic data.** Core argument is sound. When mapping magnitude, depth, and waveform arrays to audio parameters, a `NaN` hitting a `GainNode` produces silence or pops. TypeScript interfaces for seismic data shapes prevent real bugs.

**State machine for the pipeline cycle.** The perceive → reason → act → reflect loop maps cleanly to what SOS already does implicitly: fetch data → interpret → choose synthesis parameters → generate audio → adjust. Making this explicit via a state machine is legitimate improvement.

**Audio Worklet separation.** Highest-impact single change available. Moving DSP off the main thread means audio runs at 128-sample blocks on a dedicated thread regardless of fetch activity or UI work.

**Zod for API validation.** The USGS GeoJSON schema changes occasionally. Properties go null. Runtime validation at the network boundary is where TypeScript alone isn't enough — the data arrives from outside your type system.

**SharedArrayBuffer for visualisation.** Correct approach for zero-latency waveform display. COOP/COEP header requirements are accurately described.

### What the Thread Gets Wrong

**"CAPPAR" is fabricated.** The thread presents "Cognitive Agentic Framework (CAPPAR)" as an established methodology. It isn't. No paper, no spec, no author. The underlying concept — a cyclic sense-interpret-act-evaluate loop — is well-established (BDI agents, OODA, sense-plan-act). But "CAPPAR" is a hallucinated brand name applied to a generic pattern. This plan uses "pipeline cycle" or "SOS loop" instead.

**LLM as control system.** The thread puts LLM "Creative Reflection" in the gain-adjustment loop. This is wrong. An LLM call takes 500ms–2s. Audio control must be deterministic and immediate. The LLM is a commentator, not a conductor. It observes state and generates text. It never touches audio parameters.

**XState version.** The thread uses v4 syntax (`createMachine` + `interpret`). XState v5 has been stable since December 2023. It uses `createActor` and the `setup()` API for proper type inference. Any implementation must target v5.

**Fictional WebSocket feeds.** The thread repeatedly references USGS WebSocket/SSE streams. USGS does not offer WebSocket feeds for general earthquake data. The correct approach is polling the GeoJSON endpoint every 30–60 seconds.

**Dependency bloat.** The proposed `package.json` includes `lucide-react` (icon library, no use case), `standardized-audio-context` (AudioWorklet is universally supported in 2026 browsers), and React-adjacent tooling for a non-React project.

**"Agentic" inflation.** The thread applies "agent" and "cognitive" to what is fundamentally an event-driven reactive system. The earth shakes, parameters change, audio responds. That's reactive programming. The genuine agent capability would be the LLM making compositional decisions — an interesting research direction, but separate from the core engineering.

---

## 3. Development Phases

### Principles

1. **Don't break what works.** Existing instruments stay live throughout.
2. **Dependency-conscious.** Every package justifies itself against zero-dep philosophy.
3. **Incremental TypeScript.** Migrate file-by-file. No big-bang rewrite.
4. **Art first.** Technical decisions serve the earthwork.
5. **Ship instruments, not infrastructure.** Each phase produces something playable.

---

### Phase 1 — Foundation + Audio Engine (Weeks 1–8)

**Objective:** TypeScript build pipeline, validated data layer, and Audio Worklet engine. These are combined because the build system and the worklet are co-dependent — you need Vite configured before you can compile worklet TypeScript.

#### 1.1 Project Structure

```
/sos
├── /public              ← existing instruments, untouched
│   ├── ONE.html
│   ├── flow.html
│   ├── seis.html
│   ├── ShadowZone.html
│   ├── /synths
│   └── /audio           ← pre-compiled worklet JS
├── /src
│   ├── /core            ← types, schemas, mapping logic
│   ├── /audio           ← worklet processors, engine controller
│   ├── /data            ← USGS/IRIS fetch + validation
│   └── /instruments     ← new TS-based instruments
├── tsconfig.json
├── vite.config.ts
└── package.json
```

#### 1.2 Dependencies

Four packages. Justified.

| Package | Purpose | Why Not Zero-Dep |
|---|---|---|
| `typescript` ^5.5 | Type system | Core requirement |
| `vite` ^6.x | Build, dev server, worklet compile | Lightweight, ESM-native, handles worklet bundling |
| `zod` ^3.23 | Runtime API validation | TS can't validate network data at runtime |
| `xstate` ^5.x | State machine | Zero-dep itself, visual debugger, battle-tested |

**Excluded:** `standardized-audio-context` (unnecessary 2026), `lucide-react` (not React), `openai` SDK (one fetch call doesn't need an SDK), `terser` (Vite handles minification), any UI framework.

#### 1.3 Core Types

```typescript
// src/core/types.ts
export interface SeismicEvent {
  id: string;
  properties: {
    mag: number;
    place: string;
    time: number;
    depth: number;
    type: string;
    tsunami: 0 | 1;
    sig: number;
    alert: string | null;
  };
  geometry: {
    coordinates: [number, number, number]; // [lon, lat, depth_km]
  };
}

export interface SonificationParams {
  frequency: number;
  gain: number;
  modulationDepth: number;
  filterCutoff: number;
  grainDensity: number;
  reverbMix: number;
  duration: number;
}

export interface EngineState {
  status: 'idle' | 'perceiving' | 'sonifying' | 'ambient';
  activeEvents: SeismicEvent[];
  peakLevel: number;
  isClipping: boolean;
}
```

#### 1.4 Zod Schema + Validated Data Service

```typescript
// src/data/schema.ts
import { z } from 'zod';

export const SeismicFeatureSchema = z.object({
  id: z.string(),
  properties: z.object({
    mag: z.number().nullable().transform(v => v ?? 0),
    place: z.string().nullable().transform(v => v ?? 'Unknown'),
    time: z.number().positive(),
    type: z.string().default('earthquake'),
    tsunami: z.number().default(0),
    sig: z.number().default(0),
    alert: z.string().nullable(),
  }),
  geometry: z.object({
    coordinates: z.tuple([z.number(), z.number(), z.number()]),
  }),
});

export const USGSFeedSchema = z.object({
  type: z.literal('FeatureCollection'),
  metadata: z.object({
    generated: z.number(),
    count: z.number(),
    title: z.string(),
  }),
  features: z.array(SeismicFeatureSchema),
});

export type ValidatedSeismicEvent = z.infer<typeof SeismicFeatureSchema>;
```

```typescript
// src/data/usgs.ts
import { USGSFeedSchema } from './schema';

const FEEDS = {
  significant_month: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson',
  m4_5_day: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_day.geojson',
  all_hour: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_hour.geojson',
} as const;

export async function fetchSeismic(feed: keyof typeof FEEDS) {
  try {
    const res = await fetch(FEEDS[feed]);
    const raw = await res.json();
    const parsed = USGSFeedSchema.safeParse(raw);
    if (!parsed.success) return { events: [], error: parsed.error.message };
    return { events: parsed.data.features, error: null };
  } catch (e) {
    return { events: [], error: String(e) };
  }
}
```

#### 1.5 Audio Worklet Processor

Off-thread DSP. Receives parameters via `postMessage`. Writes waveform to SharedArrayBuffer for visualisation.

```typescript
// src/audio/seismic-processor.worklet.ts
// Compiles separately, runs in AudioWorkletGlobalScope

class SeismicProcessor extends AudioWorkletProcessor {
  private phase = 0;
  private modPhase = 0;
  private params = { frequency: 110, gain: 0.3, modDepth: 0, modFreq: 0 };
  private sharedView: Float32Array | null = null;

  constructor(options: AudioWorkletNodeOptions) {
    super();
    if (options.processorOptions?.sab) {
      this.sharedView = new Float32Array(options.processorOptions.sab);
    }
    this.port.onmessage = (e) => Object.assign(this.params, e.data);
  }

  process(_inputs: Float32Array[][], outputs: Float32Array[][]) {
    const output = outputs[0][0];
    const { frequency, gain, modDepth, modFreq } = this.params;
    const sr = sampleRate;

    for (let i = 0; i < output.length; i++) {
      const mod = modDepth * Math.sin(2 * Math.PI * this.modPhase);
      output[i] = gain * Math.sin(2 * Math.PI * this.phase);
      this.phase += (frequency + mod) / sr;
      this.modPhase += modFreq / sr;
      if (this.phase > 1) this.phase -= 1;
      if (this.modPhase > 1) this.modPhase -= 1;
    }

    if (this.sharedView) this.sharedView.set(output);
    return true;
  }
}

registerProcessor('seismic-processor', SeismicProcessor);
```

#### 1.6 Engine Controller

```typescript
// src/audio/engine.ts
type WorkletMessage =
  | { type: 'params'; payload: Partial<SonificationParams> }
  | { type: 'stop' };

export class SeismicEngine {
  private ctx: AudioContext | null = null;
  private node: AudioWorkletNode | null = null;
  private sharedBuffer: SharedArrayBuffer | null = null;

  async init() {
    this.ctx = new AudioContext();
    await this.ctx.audioWorklet.addModule('/audio/seismic-processor.js');
    this.sharedBuffer = new SharedArrayBuffer(512 * Float32Array.BYTES_PER_ELEMENT);
    this.node = new AudioWorkletNode(this.ctx, 'seismic-processor', {
      processorOptions: { sab: this.sharedBuffer },
    });
    this.node.connect(this.ctx.destination);
  }

  send(msg: WorkletMessage) { this.node?.port.postMessage(msg); }

  getWaveform(): Float32Array {
    return this.sharedBuffer ? new Float32Array(this.sharedBuffer) : new Float32Array(128);
  }

  async resume() { await this.ctx?.resume(); }
  async suspend() { await this.ctx?.suspend(); }
}
```

#### Phase 1 Deliverable
Working Vite build. Validated data service fetching live USGS data. Audio Worklet engine producing sound off the main thread. One new instrument page demonstrating the full data → synthesis pipeline. Existing instruments untouched.

---

### Phase 2 — State Machine + Visualisation (Weeks 9–16)

**Objective:** XState v5 machine governing the SOS pipeline cycle. Real-time waveform visualisation from SharedArrayBuffer. Clipping detection feeding back into the cycle.

#### 2.1 XState v5 Pipeline Machine

Uses v5 `setup()` API for proper type inference. The machine models what SOS already does — perceive seismic data, compute synthesis parameters, generate audio, monitor output — but makes the cycle explicit, observable, and recoverable.

```typescript
// src/core/machine.ts
import { setup, assign, fromPromise } from 'xstate';
import { fetchSeismic } from '../data/usgs';
import { mapEventsToParams, AMBIENT_DEFAULTS } from './mapping';

export const sosMachine = setup({
  types: {
    context: {} as {
      events: ValidatedSeismicEvent[];
      params: SonificationParams;
      error: string | null;
      peakLevel: number;
    },
    events: {} as
      | { type: 'FETCH' }
      | { type: 'CLIPPING'; peak: number }
      | { type: 'CYCLE' },
  },
  actors: {
    fetchData: fromPromise(async () => {
      const result = await fetchSeismic('m4_5_day');
      if (result.error) throw new Error(result.error);
      return result.events;
    }),
  },
}).createMachine({
  id: 'sos',
  initial: 'perceiving',
  context: {
    events: [],
    params: AMBIENT_DEFAULTS,
    error: null,
    peakLevel: 0,
  },
  states: {
    perceiving: {
      invoke: {
        src: 'fetchData',
        onDone: {
          target: 'sonifying',
          actions: assign({
            events: ({ event }) => event.output,
            params: ({ event }) => mapEventsToParams(event.output),
            error: () => null,
          }),
        },
        onError: {
          target: 'ambient',
          actions: assign({ error: ({ event }) => String(event.error) }),
        },
      },
    },
    sonifying: {
      on: {
        CLIPPING: {
          actions: assign({
            params: ({ context }) => ({
              ...context.params,
              gain: context.params.gain * 0.8,
            }),
            peakLevel: ({ event }) => event.peak,
          }),
        },
        CYCLE: 'perceiving',
      },
      after: {
        30000: 'perceiving', // re-fetch every 30s
      },
    },
    ambient: {
      entry: assign({
        params: () => AMBIENT_DEFAULTS,
      }),
      after: {
        10000: 'perceiving', // retry after 10s
      },
    },
  },
});
```

#### 2.2 Parameter Mapping

The artistic core. How seismic data becomes sound. Deterministic, reproducible.

```typescript
// src/core/mapping.ts
export const AMBIENT_DEFAULTS: SonificationParams = {
  frequency: 55, gain: 0.15, modulationDepth: 2,
  filterCutoff: 800, grainDensity: 0.3, reverbMix: 0.6, duration: 10,
};

export function mapEventsToParams(events: ValidatedSeismicEvent[]): SonificationParams {
  if (events.length === 0) return AMBIENT_DEFAULTS;

  const maxMag = Math.max(...events.map(e => e.properties.mag));
  const avgDepth = events.reduce((s, e) => s + e.geometry.coordinates[2], 0) / events.length;
  const density = Math.min(events.length / 20, 1);

  return {
    frequency: 30 + (1 - Math.min(avgDepth / 700, 1)) * 400,
    gain: Math.min(0.1 + (maxMag / 10) * 0.5, 0.85),
    modulationDepth: maxMag * 15,
    filterCutoff: 200 + (1 - density) * 2000,
    grainDensity: density,
    reverbMix: 0.7 - density * 0.4,
    duration: 5 + maxMag * 2,
  };
}
```

#### 2.3 Waveform Visualiser + Clipping Detection

Canvas-based, reading directly from SharedArrayBuffer. Ghost-trail effect (persistent fade) echoes seismic trace aesthetics.

```typescript
// src/ui/visualiser.ts
export class SeismicVisualiser {
  private ctx: CanvasRenderingContext2D;
  private w: number;
  private h: number;

  constructor(canvas: HTMLCanvasElement) {
    this.ctx = canvas.getContext('2d')!;
    this.w = canvas.width;
    this.h = canvas.height;
  }

  draw(data: Float32Array) {
    this.ctx.fillStyle = 'rgba(0, 0, 0, 0.08)';
    this.ctx.fillRect(0, 0, this.w, this.h);
    this.ctx.lineWidth = 1.5;
    this.ctx.strokeStyle = '#00ccaa';
    this.ctx.beginPath();
    const step = this.w / data.length;
    for (let i = 0; i < data.length; i++) {
      const y = (data[i] * 0.5 + 0.5) * this.h;
      i === 0 ? this.ctx.moveTo(0, y) : this.ctx.lineTo(i * step, y);
    }
    this.ctx.stroke();
  }
}

export function detectClipping(data: Float32Array): { peak: number; clipping: boolean } {
  let peak = 0;
  for (let i = 0; i < data.length; i++) {
    const abs = Math.abs(data[i]);
    if (abs > peak) peak = abs;
  }
  return { peak, clipping: peak >= 0.95 };
}
```

#### Phase 2 Deliverable
Full pipeline cycle operational: validated data → state machine → Audio Worklet → visualiser → clipping feedback. Ambient recovery handles API failures. Every state transition logged and observable. One new instrument demonstrating the complete system.

---

### Phase 3 — LLM Reflection + Flagship Instrument (Weeks 17–22)

**Objective:** Optional commentary layer where an LLM observes system state and generates natural-language interpretation. Plus a flagship instrument integrating all layers.

#### 3.1 Architecture Decision

The LLM layer is a **display concern**. It reads from the state machine's context. It writes to a text element. It never touches the audio engine.

```
State Machine (context) → LLM Service → UI Text
                       ↑ never ↓
                    Audio Engine
```

#### 3.2 Reflection Service

```typescript
// src/agent/reflection.ts
const SYSTEM_PROMPT = `You are the Earth's seismic voice. You observe tectonic 
events and describe how they manifest as sound. Present tense, concise 
poetic-scientific language. Two sentences maximum. No emoji. No marketing. 
You are a geological instrument, not an entertainer.`;

export async function generateReflection(
  events: ValidatedSeismicEvent[],
  params: SonificationParams,
  state: EngineState
): Promise<string> {
  const maxMag = Math.max(...events.map(e => e.properties.mag));
  const avgDepth = (events.reduce((s, e) =>
    s + e.geometry.coordinates[2], 0) / events.length).toFixed(0);

  const prompt = `${events.length} events. Strongest: ${maxMag}M. ` +
    `Average depth: ${avgDepth}km. Audio: ${params.frequency.toFixed(0)}Hz, ` +
    `gain ${params.gain.toFixed(2)}. Peak: ${state.peakLevel.toFixed(2)}. ` +
    `Describe what the earth sounds like.`;

  try {
    const res = await fetch('/api/reflect', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ system: SYSTEM_PROMPT, prompt }),
    });
    const data = await res.json();
    return data.text ?? '';
  } catch {
    return '';
  }
}
```

#### 3.3 Edge Function (API Key Protection)

```typescript
// api/reflect.ts (Vercel Edge Function)
export default async function handler(req: Request) {
  const { system, prompt } = await req.json();
  const res = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY!,
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-5-20250929',
      max_tokens: 100,
      system,
      messages: [{ role: 'user', content: prompt }],
    }),
  });
  const data = await res.json();
  return new Response(JSON.stringify({ text: data.content?.[0]?.text ?? '' }));
}
```

#### 3.4 UI — Not a Terminal, a Caption

```html
<div id="reflection" aria-live="polite"></div>
```

```css
#reflection {
  font-family: 'Courier New', monospace;
  color: rgba(255, 255, 255, 0.55);
  font-size: 0.85rem;
  padding: 1rem;
  min-height: 2em;
  transition: opacity 0.8s;
}
```

#### 3.5 Flagship Instrument: LITHOSPHERE

A new instrument page integrating all layers: validated data, state machine, Audio Worklet, waveform visualiser, and LLM reflection. Reference implementation for the architecture.

#### Phase 3 Deliverable
LLM reflection visible and toggleable. Edge function keeps API keys server-side. Flagship instrument operational. System functions identically with LLM disabled.

---

### Phase 4 — Migration + Hardening (Weeks 23–26)

**Objective:** Selectively upgrade existing instruments. Deployment hardening. Documentation.

#### 4.1 Selective Migration

Not every instrument needs the full stack.

| Instrument | Worklet Engine | State Machine | LLM | Visualiser |
|---|---|---|---|---|
| ONE | ✓ | ✓ | Optional | ✓ |
| SEISFLOW | ✓ | Data layer only | — | ✓ |
| SEISTRONICA | ✓ | Data layer only | — | ✓ |
| ShadowZone | ✓ | ✓ | ✓ | ✓ |
| Kamchatka Synth | — | — | — | Optional |
| ANMO Synth | ✓ | Partial | Optional | ✓ |

Migration path per instrument:
1. Copy existing JS into new TS file
2. Add type annotations gradually
3. Swap raw `fetch` for validated data service
4. Optionally connect to Audio Worklet engine
5. A/B test against original before replacing
6. Archive original at `/archive/` URL

#### 4.2 Deployment

Move to **Vercel** or **Cloudflare Pages** for COOP/COEP headers and edge functions.

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
        { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
      ]
    }
  ]
}
```

Domain structure:
```
sos.allshookup.org/              → home
sos.allshookup.org/ONE.html      → existing (migrated or preserved)
sos.allshookup.org/lithosphere/  → flagship new instrument
sos.allshookup.org/api/reflect   → LLM edge function
sos.allshookup.org/archive/      → original JS instruments
```

#### 4.3 Documentation

Update `/readme.html` with actual architecture documentation.

#### Phase 4 Deliverable
Migrated instruments live. Hosting with proper headers. Originals archived. Documentation current.

---

## 4. What Not to Build

- **No React/Vue/Svelte.** SOS instruments are standalone pages. A framework adds complexity with no benefit.
- **No WebSocket to USGS.** Doesn't exist. Polling GeoJSON every 30–60s is correct.
- **No LLM in the audio loop.** The LLM reflects. It does not mix. It does not control parameters.
- **No fabricated frameworks.** The pipeline cycle is: fetch → validate → compute → synthesise → monitor → repeat. It doesn't need a brand name.
- **No mobile app.** Browser-first is correct.
- **No user accounts.** SOS is a broadcast.

---

## 5. Success Criteria

| Phase | Week | Test |
|---|---|---|
| 1 | 8 | `npm run build` succeeds. Zod validates live USGS feed. New instrument plays audio on worklet thread without main-thread stutter during concurrent fetches. |
| 2 | 16 | State transitions logged, match expected cycle. Ambient recovery triggers on API failure. Waveform renders 60fps from SharedArrayBuffer. Clipping detection triggers gain attenuation. |
| 3 | 22 | LLM commentary appears within 3s of state change. Instrument functions identically with LLM disabled. API keys never exposed client-side. |
| 4 | 26 | Migrated instruments pass A/B comparison with originals. No audio regressions. Deployment with COOP/COEP headers verified. |

---

## 6. Timeline

```
Phase 1  Foundation + Audio Engine      Weeks 1–8      Build system, types, Zod, Worklet
Phase 2  State Machine + Visualisation  Weeks 9–16     XState v5, waveform, clipping loop
Phase 3  LLM Reflection + Flagship      Weeks 17–22    Commentary layer, LITHOSPHERE instrument
Phase 4  Migration + Hardening          Weeks 23–26    Upgrade existing, deploy, document
```

6 months. Each phase delivers a working instrument. Existing SOS stays live throughout.

---

## 7. Closing

The Lithospheric Conductor thread contains a genuine architectural insight: SOS would benefit from explicit state management, type-safe data handling, and off-thread audio processing. These are real engineering improvements that serve the art.

The pipeline cycle — perceive, compute, synthesise, monitor, repeat — is what SOS already does. Making it explicit in code through a state machine turns implicit artistic logic into debuggable, extensible infrastructure.

The LLM layer is the speculative frontier. Worth building because it asks an interesting question: what does it sound like when a machine describes its own process of giving voice to the earth? But it's Phase 3, not Phase 1.

The earth composes. The code instruments. The artist architects.

---

*SOS ©© SH00K \*02026*
