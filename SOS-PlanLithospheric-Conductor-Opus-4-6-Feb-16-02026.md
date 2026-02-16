# SOS Development Plan
### Sounds of Seismic — Electronica Earthwork
**Prepared:** Mon 16 Feb 2026  
**Source Analysis:** "The Lithospheric Conductor: Architecting an Agentic Earthwork" (Google AI Studio thread, 16 Feb 2026)  
**Live Reference:** [sos.allshookup.org](https://sos.allshookup.org) · [allshookup.org/seismic-field.html](https://allshookup.org/seismic-field.html)

---

## 0. Purpose of This Document

The Lithospheric Conductor thread walks through a full conceptual architecture for upgrading SOS from its current pure-JavaScript, dependency-free browser implementation to a TypeScript-driven agentic system. It covers type safety, cognitive loops, Audio Worklets, SharedArrayBuffer visualisation, LLM integration, and deployment config.

This plan separates what's real from what's aspirational, identifies what the Gemini thread gets right, flags where it drifts into generic AI-slop territory, and lays out a phased execution path that respects the existing SOS codebase, SHOOK's artistic intent, and the practical constraints of a solo interdisciplinary practice.

---

## 1. Current State Assessment

### What Exists (Production)

| Interface | URL | Function |
|---|---|---|
| **SOS Home** | sos.allshookup.org | Landing page, tectonic data loader, navigation hub |
| **ONE** | /ONE.html | Seismic ambient electronica — single-page sonification |
| **SEISFLOW** | /flow.html | Melodic mapping of seismic wave frequencies |
| **SEISTRONICA** | /seis.html | Classic electronica blended with seismic data |
| **ShadowZone** | /ShadowZone.html | Three-layer granular synthesis with scale/root/stretch controls |
| **Kamchatka M8.8 Synth** | /synths/Kamchatka-8-8-Synth.html | Event-specific sonification (2025 Kamchatka earthquake) |
| **ANMO FM Synth** | /synths/ANMO-FM-Synth.html | Station-specific AI synthesizer (ANMO seismic station) |
| **Seismic Field** | allshookup.org/seismic-field.html | M6+ global earthquake archive, data back to 1900 |
| **README** | /readme.html | Full conceptual framework, technical docs |

### Technical Stack (Current)

- Pure JavaScript — zero external dependencies
- Web Audio API (native browser)
- USGS GeoJSON feeds + EarthScope/IRIS MiniSEED (BHZ channel)
- Custom Earthquake Sound Engine (ESE)
- Algorithmic granular synthesis
- LLM integration referenced in README but extent of current implementation unclear
- Hosted statically (likely GitHub Pages or equivalent)
- GitHub: [github.com/strikeslip](https://github.com/strikeslip) (5 repositories)

### What Works Well

The dependency-free philosophy is a genuine strength, not a weakness. Every synth page loads instantly, runs anywhere, and has zero supply-chain risk. The conceptual framework (Smithson's site/non-site, planetary non-site, seismic unconscious) is rigorous and original. The collection of instruments represents genuine artistic range — ambient, melodic, granular, station-specific, event-specific.

---

## 2. Lithospheric Conductor Thread — Critical Analysis

### What the Thread Gets Right

**Type safety for seismic data.** The core argument is sound: when you're mapping magnitude, depth, and waveform arrays to audio parameters, a `NaN` hitting a `GainNode` produces silence or pops. TypeScript interfaces for `SeismicEvent` data shapes would prevent real bugs that exist in any system parsing external API data.

**State machine for the cognitive loop.** The Perceive–Reason–Plan–Act–Reflect (CAPPAR) model maps cleanly to what SOS already does implicitly: fetch data → interpret it → choose synthesis parameters → generate audio → (potentially) adjust. Making this explicit via a state machine is legitimate architectural improvement, not decoration.

**Audio Worklet separation.** Moving DSP off the main thread is the single highest-impact technical change available. Right now, if a USGS fetch takes time or the browser tab is doing anything else, audio can stutter. An Audio Worklet runs on a dedicated thread at 128-sample blocks regardless of main thread activity.

**Zod for API validation.** The USGS GeoJSON schema changes occasionally. Properties go null. New fields appear. Runtime validation at the network boundary is genuinely useful — this is where TypeScript alone isn't enough because the data comes from outside your type system.

**SharedArrayBuffer for visualisation.** For real-time waveform display without postMessage latency, this is the correct approach. The thread's COOP/COEP header discussion is accurate and necessary.

### Where the Thread Drifts

**Overengineering the LLM layer.** The thread treats LLM "Creative Reflection" as a core architectural component. In reality, having an LLM narrate mixing decisions ("The weight of the Pacific plate is too heavy for these oscillators") is aesthetically interesting but not a prerequisite for anything else. It's a feature, not infrastructure. The thread conflates artistic commentary with system control — the LLM should never be in the gain-adjustment loop.

**XState version.** The thread uses XState v4 syntax (`createMachine` + `interpret`). XState v5 has been stable since December 2023 and uses `createActor` with the `setup()` API for type inference. Any implementation should target v5.

**Package bloat.** The proposed `package.json` includes `lucide-react` (an icon library) with no justification. The `standardized-audio-context` polyfill may be unnecessary in 2026 — Web Audio API and AudioWorklet support is now universal across modern browsers. Each dependency needs to justify its existence against SOS's zero-dependency philosophy.

**"Agentic" framing.** The thread applies "agent" and "cognitive" liberally to what is fundamentally an event-driven reactive system. The earth shakes, parameters change, audio responds. This is reactive programming, which is fine. Calling it "agentic" is marketing language. The actual agent capability — if pursued — would be the LLM layer making genuine compositional decisions beyond parameter mapping. That's an interesting research direction but a separate workstream.

**Vite worker/worklet bundling.** The `?worker&url` import strategy described is partially correct but glossed over. Vite's AudioWorklet support requires careful configuration — the thread's treatment is superficial. In practice, placing a pre-compiled worklet in `/public` is often more reliable than fighting the bundler.

**CRT terminal aesthetic.** The scanline/flicker CSS is generic retro-tech styling. SOS already has its own aesthetic identity. The UI layer should emerge from the existing visual language, not from a Gemini template.

---

## 3. Development Phases

### Guiding Principles

1. **Don't break what works.** The existing instruments stay live and functional throughout.
2. **Dependency-conscious.** Every new package must justify itself. Prefer zero-dep solutions where feasible.
3. **Incremental TypeScript.** Migrate file-by-file using `.ts` alongside existing `.js`. No big-bang rewrite.
4. **Art first.** Technical decisions serve the earthwork. The earth is the composer. The code is the instrument.
5. **Ship instruments, not infrastructure.** Each phase should produce a new playable instrument or a measurable improvement to an existing one.

---

### Phase 1 — Foundation (Weeks 1–4)

**Objective:** Establish TypeScript build pipeline without disrupting existing instruments.

#### 1.1 Build System

Set up Vite alongside existing static files. The existing HTML instruments remain served as-is. New TypeScript code compiles into the same deployment.

```
/sos
├── /public              ← existing instruments live here unchanged
│   ├── ONE.html
│   ├── flow.html
│   ├── seis.html
│   ├── ShadowZone.html
│   └── /synths
├── /src                 ← new TypeScript source
│   ├── /core            ← shared types, schemas, utilities
│   ├── /audio           ← Audio Worklet processors
│   ├── /data            ← USGS/IRIS data layer
│   └── /instruments     ← new TS-based instruments
├── tsconfig.json
├── vite.config.ts
└── package.json
```

#### 1.2 Core Type Definitions

Define the data contracts that matter. These aren't abstract — they map directly to USGS GeoJSON and MiniSEED structures.

```typescript
// src/core/types.ts

export interface SeismicEvent {
  id: string;
  properties: {
    mag: number;
    place: string;
    time: number;        // epoch ms
    depth: number;       // km (extracted from geometry)
    type: string;        // "earthquake", "quarry blast", etc.
    tsunami: 0 | 1;
    sig: number;         // significance score
    alert: string | null;
  };
  geometry: {
    coordinates: [number, number, number]; // [lon, lat, depth]
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

#### 1.3 Zod Schema for USGS Feed

Runtime validation at the API boundary. This catches the real-world problems: null magnitudes, missing coordinates, API format changes.

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
```

#### 1.4 Validated Data Service

Replace raw `fetch` with a validated pipeline. Graceful degradation — if validation fails, the system knows why and can respond accordingly.

```typescript
// src/data/usgs.ts
import { USGSFeedSchema, type SeismicFeature } from './schema';

const FEEDS = {
  significant_month: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson',
  m4_5_day: 'https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_day.geojson',
  m6_plus: 'https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&minmagnitude=6',
} as const;

export async function fetchSeismic(
  feed: keyof typeof FEEDS
): Promise<{ events: SeismicFeature[]; error: string | null }> {
  try {
    const res = await fetch(FEEDS[feed]);
    const raw = await res.json();
    const parsed = USGSFeedSchema.safeParse(raw);
    if (!parsed.success) {
      return { events: [], error: parsed.error.message };
    }
    return { events: parsed.data.features, error: null };
  } catch (e) {
    return { events: [], error: String(e) };
  }
}
```

#### Phase 1 Deliverable
A working Vite build that compiles TypeScript alongside existing instruments. Validated USGS data service ready for use. No user-facing changes yet.

---

### Phase 2 — Audio Worklet Engine (Weeks 5–10)

**Objective:** Move DSP off the main thread. Build a reusable synthesis processor.

This is the highest-impact technical work in the entire plan. Every existing instrument would benefit from this, and it's the prerequisite for real-time visualisation and the state machine layer.

#### 2.1 Audio Worklet Processor

The worklet runs in its own thread. It receives parameter updates via `port.postMessage` and writes audio output at 128-sample blocks. It also writes waveform data to a SharedArrayBuffer for visualisation.

```typescript
// src/audio/seismic-processor.worklet.ts
// This file compiles separately and runs in AudioWorkletGlobalScope

interface SeismicParams {
  frequency: number;
  gain: number;
  modDepth: number;
  modFreq: number;
}

class SeismicProcessor extends AudioWorkletProcessor {
  private phase = 0;
  private modPhase = 0;
  private params: SeismicParams = {
    frequency: 110,
    gain: 0.3,
    modDepth: 0,
    modFreq: 0,
  };
  private sharedView: Float32Array | null = null;

  constructor(options: AudioWorkletNodeOptions) {
    super();
    if (options.processorOptions?.sab) {
      this.sharedView = new Float32Array(options.processorOptions.sab);
    }
    this.port.onmessage = (e) => {
      Object.assign(this.params, e.data);
    };
  }

  process(_inputs: Float32Array[][], outputs: Float32Array[][]) {
    const output = outputs[0][0];
    const { frequency, gain, modDepth, modFreq } = this.params;
    const sr = sampleRate;

    for (let i = 0; i < output.length; i++) {
      const mod = modDepth * Math.sin(2 * Math.PI * this.modPhase);
      const freq = frequency + mod;
      output[i] = gain * Math.sin(2 * Math.PI * this.phase);
      this.phase += freq / sr;
      this.modPhase += modFreq / sr;
      if (this.phase > 1) this.phase -= 1;
      if (this.modPhase > 1) this.modPhase -= 1;
    }

    // Write to shared buffer for visualiser
    if (this.sharedView) {
      this.sharedView.set(output);
    }

    return true;
  }
}

registerProcessor('seismic-processor', SeismicProcessor);
```

#### 2.2 Engine Controller (Main Thread)

Type-safe message protocol between the main thread "brain" and the worklet "instrument."

```typescript
// src/audio/engine.ts
import type { SonificationParams } from '../core/types';

type WorkletMessage =
  | { type: 'params'; payload: Partial<SonificationParams> }
  | { type: 'stop' };

export class SeismicEngine {
  private ctx: AudioContext | null = null;
  private node: AudioWorkletNode | null = null;
  private sharedBuffer: SharedArrayBuffer | null = null;

  async init() {
    this.ctx = new AudioContext();
    // Worklet file placed in /public for reliable loading
    await this.ctx.audioWorklet.addModule('/audio/seismic-processor.js');

    // SharedArrayBuffer for zero-latency waveform reads
    this.sharedBuffer = new SharedArrayBuffer(512 * Float32Array.BYTES_PER_ELEMENT);

    this.node = new AudioWorkletNode(this.ctx, 'seismic-processor', {
      processorOptions: { sab: this.sharedBuffer },
    });
    this.node.connect(this.ctx.destination);
  }

  send(msg: WorkletMessage) {
    this.node?.port.postMessage(msg);
  }

  getWaveform(): Float32Array {
    return this.sharedBuffer
      ? new Float32Array(this.sharedBuffer)
      : new Float32Array(128);
  }

  async resume() { await this.ctx?.resume(); }
  async suspend() { await this.ctx?.suspend(); }
}
```

#### 2.3 COOP/COEP Headers

SharedArrayBuffer requires Cross-Origin Isolation. Configuration depends on hosting.

For **Vercel** (`vercel.json`):
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

For **GitHub Pages** (no custom headers), use a Service Worker to inject headers or consider migrating hosting.

#### Phase 2 Deliverable
One new instrument page built on the Audio Worklet engine. Smooth audio regardless of main thread activity. Waveform data accessible via SharedArrayBuffer. Existing instruments untouched.

---

### Phase 3 — State Machine Layer (Weeks 11–16)

**Objective:** Implement the CAPPAR cognitive loop using XState v5 as an explicit state machine governing the data-to-sound pipeline.

This is where the Lithospheric Conductor's central concept gets implemented — but correctly, using v5 syntax.

#### 3.1 XState v5 Machine

```typescript
// src/core/machine.ts
import { setup, assign, fromPromise } from 'xstate';
import { fetchSeismic } from '../data/usgs';
import type { SeismicEvent, SonificationParams, EngineState } from './types';

export const sosMachine = setup({
  types: {
    context: {} as {
      events: SeismicEvent[];
      params: SonificationParams;
      reflection: string;
      error: string | null;
      peakLevel: number;
    },
    events: {} as
      | { type: 'FETCH' }
      | { type: 'DATA_RECEIVED'; events: SeismicEvent[] }
      | { type: 'VALIDATION_ERROR'; error: string }
      | { type: 'SONIFY' }
      | { type: 'AUDIO_COMPLETE' }
      | { type: 'CLIPPING_DETECTED'; peak: number }
      | { type: 'CYCLE' },
  },
  actors: {
    fetchSeismicData: fromPromise(async () => {
      const result = await fetchSeismic('m4_5_day');
      if (result.error) throw new Error(result.error);
      return result.events;
    }),
  },
  actions: {
    assignEvents: assign({
      events: (_, params: { events: SeismicEvent[] }) => params.events,
      error: () => null,
    }),
    assignError: assign({
      error: (_, params: { error: string }) => params.error,
    }),
    computeParams: assign({
      params: ({ context }) => mapEventsToParams(context.events),
    }),
    attenuate: assign({
      params: ({ context }) => ({
        ...context.params,
        gain: context.params.gain * 0.8,
      }),
    }),
    setAmbient: assign({
      params: () => AMBIENT_DEFAULTS,
      reflection: () => 'Sensor gap — sustaining internal resonance.',
    }),
  },
}).createMachine({
  id: 'sos',
  initial: 'perceiving',
  context: {
    events: [],
    params: AMBIENT_DEFAULTS,
    reflection: '',
    error: null,
    peakLevel: 0,
  },
  states: {
    perceiving: {
      invoke: {
        src: 'fetchSeismicData',
        onDone: {
          target: 'reasoning',
          actions: assign({
            events: ({ event }) => event.output,
          }),
        },
        onError: {
          target: 'ambient',
          actions: assign({
            error: ({ event }) => String(event.error),
          }),
        },
      },
    },
    reasoning: {
      always: {
        target: 'acting',
        actions: 'computeParams',
      },
    },
    acting: {
      on: {
        AUDIO_COMPLETE: 'reflecting',
      },
    },
    reflecting: {
      on: {
        CLIPPING_DETECTED: {
          target: 'acting',
          actions: 'attenuate',
        },
        CYCLE: 'perceiving',
      },
      after: {
        30000: 'perceiving', // Re-fetch every 30s
      },
    },
    ambient: {
      entry: 'setAmbient',
      after: {
        10000: 'perceiving', // Retry after 10s
      },
    },
  },
});
```

#### 3.2 Parameter Mapping Function

The core artistic logic: how seismic data becomes sound. This is where SOS's identity lives. The state machine orchestrates; this function composes.

```typescript
// src/core/mapping.ts
import type { SeismicEvent, SonificationParams } from './types';

const AMBIENT_DEFAULTS: SonificationParams = {
  frequency: 55,
  gain: 0.15,
  modulationDepth: 2,
  filterCutoff: 800,
  grainDensity: 0.3,
  reverbMix: 0.6,
  duration: 10,
};

export function mapEventsToParams(events: SeismicEvent[]): SonificationParams {
  if (events.length === 0) return AMBIENT_DEFAULTS;

  const maxMag = Math.max(...events.map(e => e.properties.mag));
  const avgDepth = events.reduce((s, e) => s + e.geometry.coordinates[2], 0) / events.length;
  const density = Math.min(events.length / 20, 1);

  return {
    // Deeper = lower frequency. Shallow crustal = higher pitch.
    frequency: 30 + (1 - Math.min(avgDepth / 700, 1)) * 400,
    // Magnitude scales gain logarithmically (matches Moment Magnitude energy scaling)
    gain: Math.min(0.1 + (maxMag / 10) * 0.5, 0.85),
    // Modulation depth increases with magnitude
    modulationDepth: maxMag * 15,
    // High event density = narrower filter (focused sound)
    filterCutoff: 200 + (1 - density) * 2000,
    // Event count maps to granular density
    grainDensity: density,
    // More events = less reverb (clarity over wash)
    reverbMix: 0.7 - density * 0.4,
    // Duration proportional to significance
    duration: 5 + maxMag * 2,
  };
}
```

#### Phase 3 Deliverable
A new instrument built on the full pipeline: validated data → state machine → Audio Worklet. The state machine is observable (every transition logged). Ambient recovery handles API failures gracefully.

---

### Phase 4 — Visualisation Layer (Weeks 17–20)

**Objective:** Real-time waveform visualisation reading directly from SharedArrayBuffer. Ghost-trail aesthetic aligned with existing SOS visual identity.

#### 4.1 Waveform Renderer

Canvas-based, reading from the shared buffer established in Phase 2. No postMessage latency. The ghost-trail (persistent fade rather than clear) is a good idea from the Gemini thread — it echoes seismic trace aesthetics.

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
    // Fade previous frame — creates persistent trace
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
```

#### 4.2 Clipping Detection

Feed back into the state machine's Reflect phase.

```typescript
// src/ui/monitor.ts
export function detectClipping(data: Float32Array): { peak: number; clipping: boolean } {
  let peak = 0;
  for (let i = 0; i < data.length; i++) {
    const abs = Math.abs(data[i]);
    if (abs > peak) peak = abs;
  }
  return { peak, clipping: peak >= 0.95 };
}
```

#### Phase 4 Deliverable
Waveform visualisation running on existing and new instruments. Clipping detection feeds the state machine's reflection cycle. Visual identity consistent with SOS aesthetic.

---

### Phase 5 — LLM Reflection Layer (Weeks 21–26)

**Objective:** Optional creative commentary layer. The LLM observes the system's state and generates natural language interpretations. Strictly read-only — it does not control audio parameters.

This is the Lithospheric Conductor's most distinctive proposal. Implemented correctly, it's genuinely interesting: a machine narrating its own compositional process in geological language. Implemented wrong, it's a chatbot bolted onto a synthesiser.

#### 5.1 Architecture Decision

The LLM layer is a **display concern**, not a control concern. It reads from the state machine's context. It writes to a text element in the UI. It never writes to the audio engine.

```
State Machine (context) → LLM Service → UI Text Display
                       ↑ never ↓
                    Audio Engine
```

#### 5.2 Reflection Service

```typescript
// src/agent/reflection.ts

interface ReflectionInput {
  events: SeismicEvent[];
  params: SonificationParams;
  engineState: EngineState;
}

const SYSTEM_PROMPT = `You are the Earth's seismic voice. You observe tectonic 
events and describe how they manifest as sound. Speak in present tense, concise 
poetic-scientific language. One to two sentences maximum. No emoji. No marketing 
language. You are a geological instrument, not an entertainer.`;

export async function generateReflection(
  input: ReflectionInput,
  provider: 'anthropic' | 'ollama' = 'anthropic'
): Promise<string> {
  const baseURL = provider === 'ollama'
    ? 'http://localhost:11434/v1'
    : 'https://api.anthropic.com/v1';

  const prompt = `Current seismic state: ${input.events.length} events detected.
Strongest: ${Math.max(...input.events.map(e => e.properties.mag))}M.
Average depth: ${(input.events.reduce((s, e) => s + e.geometry.coordinates[2], 0) / input.events.length).toFixed(0)}km.
Audio: frequency=${input.params.frequency.toFixed(0)}Hz, gain=${input.params.gain.toFixed(2)}.
Engine status: ${input.engineState.status}.
Peak level: ${input.engineState.peakLevel.toFixed(2)}.
Describe what the earth sounds like right now.`;

  // Implementation depends on provider — use edge function in production
  // to keep API keys server-side
  try {
    const response = await fetch(`${baseURL}/messages`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: provider === 'ollama' ? 'llama3' : 'claude-sonnet-4-5-20250929',
        max_tokens: 100,
        system: SYSTEM_PROMPT,
        messages: [{ role: 'user', content: prompt }],
      }),
    });
    const data = await response.json();
    return data.content?.[0]?.text ?? '';
  } catch {
    return ''; // Silence on failure — the music speaks for itself
  }
}
```

#### 5.3 UI Integration

Minimal. A single text line that updates periodically. Not a terminal. Not a chatbot. A caption.

```html
<div id="reflection" aria-live="polite"></div>
```

```css
#reflection {
  font-family: 'Courier New', monospace;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.85rem;
  padding: 1rem;
  min-height: 2em;
  transition: opacity 0.5s;
}
```

#### Phase 5 Deliverable
LLM reflection visible on new instruments. Toggle-able by user. Works with Anthropic API (production) or local Ollama (development). Graceful degradation — if the LLM is unavailable, the instrument is unaffected.

---

### Phase 6 — Migration & Integration (Weeks 27–32)

**Objective:** Selectively upgrade existing instruments to use the new engine components where beneficial.

Not every instrument needs the full stack. The approach is surgical.

| Instrument | Audio Worklet | State Machine | LLM Reflection | Visualiser |
|---|---|---|---|---|
| ONE | Yes | Yes | Optional | Yes |
| SEISFLOW | Yes | Partial (data layer only) | No | Yes |
| SEISTRONICA | Yes | Partial (data layer only) | No | Yes |
| ShadowZone | Yes | Yes | Yes | Yes |
| Kamchatka Synth | No (event-specific, static data) | No | No | Optional |
| ANMO Synth | Yes | Partial | Optional | Yes |

For each instrument, the migration path is:

1. Copy existing JS logic into a new TS file
2. Add type annotations (gradual — `any` is acceptable as interim)
3. Swap raw `fetch` for validated data service
4. Optionally connect to Audio Worklet engine
5. Test extensively before replacing the live version
6. Keep the original JS version accessible at an archive URL

---

## 4. Dependency Manifest

Minimal. Justified.

| Package | Version | Purpose | Justification |
|---|---|---|---|
| `typescript` | ^5.5 | Type system | Core requirement |
| `vite` | ^6.x | Build, dev server, worklet bundling | Lightweight, fast, ESM-native |
| `zod` | ^3.23 | Runtime API validation | TypeScript alone can't validate network data |
| `xstate` | ^5.x | State machine (CAPPAR loop) | Zero-dep, battle-tested, visual debugger |

**Not included and why:**

- `standardized-audio-context` — AudioWorklet is universally supported in 2026 browsers. Not needed.
- `lucide-react` — SOS is not a React app. No icon library needed.
- `react` / any framework — SOS instruments are self-contained HTML pages. Vanilla TS + DOM is appropriate.
- `openai` SDK — The LLM integration is a single `fetch` call. An SDK for one endpoint is overhead.
- `terser` — Vite includes minification. Separate terser not needed.

---

## 5. Deployment

### Hosting Recommendation

Move from GitHub Pages to **Vercel** or **Cloudflare Pages**. Both support custom headers (required for SharedArrayBuffer), edge functions (required for secure LLM API proxying), and automatic HTTPS.

### Deployment Configuration

```
vercel.json:
- COOP/COEP headers on all routes
- /api/reflect edge function for LLM proxy
- Environment variables for API keys
- Existing static HTML instruments served from /public
```

### Domain Structure

```
sos.allshookup.org/              → SOS home (existing)
sos.allshookup.org/ONE.html      → existing instruments (preserved)
sos.allshookup.org/v2/           → new TypeScript instruments
sos.allshookup.org/api/reflect   → LLM edge function
```

---

## 6. What Not to Build

Explicit exclusions to prevent scope drift.

- **No React/Vue/Svelte.** SOS instruments are standalone HTML pages. This is correct. A framework adds complexity, bundle size, and conceptual overhead with no benefit.
- **No real-time WebSocket to USGS.** USGS does not offer a WebSocket feed for general earthquake data. The Gemini thread mentions WebSocket/SSE repeatedly but this isn't how USGS data works. Polling the GeoJSON feed every 30–60 seconds is the correct approach.
- **No LLM in the audio control loop.** The LLM reflects. It does not mix. Latency alone (500ms–2s for an API call) makes this impractical, but more importantly, the artistic mapping from seismic data to sound parameters should be deterministic and reproducible. The LLM is a commentator, not a conductor.
- **No mobile app.** Browser-first is correct. Progressive Web App features (offline caching, installability) could be added later but are not priority.
- **No user accounts or social features.** SOS is a broadcast. The earth speaks. People listen.

---

## 7. Success Criteria

Each phase has a binary test:

| Phase | Test |
|---|---|
| 1 — Foundation | `npm run build` produces valid output. Zod schema validates live USGS feed without errors. |
| 2 — Audio Worklet | New instrument plays audio without main-thread stuttering during simultaneous fetch operations. |
| 3 — State Machine | State transitions are logged and match expected CAPPAR sequence. Ambient recovery triggers on API failure. |
| 4 — Visualisation | Waveform renders at 60fps from SharedArrayBuffer while audio plays. Clipping detection triggers state transition. |
| 5 — LLM Reflection | Commentary appears within 3 seconds of state change. Instrument functions identically with LLM disabled. |
| 6 — Migration | All migrated instruments pass A/B comparison with originals. No audio regressions. |

---

## 8. Timeline Summary

```
Phase 1  Foundation          Weeks 1–4     Build system, types, schemas
Phase 2  Audio Worklet       Weeks 5–10    Off-thread DSP, SharedArrayBuffer
Phase 3  State Machine       Weeks 11–16   XState v5 CAPPAR loop
Phase 4  Visualisation       Weeks 17–20   Real-time waveform, clipping monitor
Phase 5  LLM Reflection      Weeks 21–26   Optional commentary layer
Phase 6  Migration           Weeks 27–32   Selective upgrade of existing instruments
```

Total: ~8 months at sustainable pace for a solo practitioner balancing art, finance, and development.

Phases 1–3 are the structural core. Phases 4–6 are enhancements that each independently add value.

---

## 9. Final Note

The Lithospheric Conductor thread contains a genuine architectural insight buried under layers of AI-generated enthusiasm: SOS would benefit from explicit state management, type-safe data handling, and off-thread audio processing. These are real engineering improvements that serve the art.

The cognitive loop (CAPPAR) is a useful mental model for understanding what SOS already does. Making it explicit in code — through a state machine with observable transitions — turns implicit artistic logic into debuggable, extensible infrastructure.

The LLM layer is the speculative frontier. It's worth building because it asks an interesting question: what does it sound like when a machine describes its own process of giving voice to the earth? That's a question worth asking. But it's Phase 5, not Phase 1.

The earth composes. The code instruments. The artist architects.

Build accordingly.

---

*SOS ©© SH00K \*02026*