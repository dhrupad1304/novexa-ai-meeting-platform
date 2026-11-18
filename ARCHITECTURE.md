# Novexa architecture notes

## 1. Real-time ASR flow

```mermaid
sequenceDiagram
    participant UI as Meeting UI
    participant Audio as Browser AudioContext
    participant Worklet as AudioWorklet processor
    participant STTAPI as Next.js /api/watson-stt
    participant IBM as IBM Watson Speech to Text

    UI->>Audio: Provide meeting MediaStream
    Audio->>Worklet: Audio samples
    UI->>STTAPI: Request temporary STT token
    STTAPI->>IBM: Request token using server-side API key
    IBM-->>STTAPI: Temporary access token
    STTAPI-->>UI: Token and WebSocket URL
    UI->>IBM: Open recognition WebSocket
    IBM-->>UI: WebSocket ready
    UI->>IBM: Start message (recognition config)
    Worklet-->>UI: Int16 PCM ArrayBuffer chunks
    UI->>IBM: Binary audio chunks
    IBM-->>UI: Interim and final transcript messages
    UI->>UI: Update transcript callbacks / meeting UI
```

Implementation references:

- `src/lib/watsonSTT.js` — token retrieval, WebSocket lifecycle, recognition configuration, audio capture, and transcript callbacks.
- `src/app/api/watson-stt/route.js` — server-side token endpoint.
- `public/audio-processor.js` — audio chunking and Float32-to-Int16 conversion.

## 2. Audio representation

The recognition configuration requests `audio/l16;rate=16000;channels=1`. The worklet creates `Int16Array` chunks from Float32 input and posts the underlying `ArrayBuffer` to the main thread. The client sends those buffers as binary WebSocket messages.

The `AudioContext` is constructed with a preferred `sampleRate: 16000`. Browser/device behavior should be checked at runtime; a requested sample rate is not a substitute for verifying the actual context sample rate and resampling behavior.

## 3. AI inference flow

```mermaid
flowchart LR
    A[Meeting transcript / user prompt] --> B[Next.js API route]
    B --> C[watsonx helper]
    C --> D[IBM IAM token endpoint]
    D --> C
    C --> E[watsonx inference API]
    E --> F[Granite model response]
    F --> G[Meeting summary / assistant response]
```

Implementation references:

- `src/lib/watsonx.js` — IAM token exchange and authenticated watsonx request helper.
- `src/app/api/meetings/` — meeting summary, chat, and auto-summary routes.
- `src/app/api/agent/route.js` — agent API route.

## 4. Data and application services

The application uses Next.js API routes and Mongoose/MongoDB for persistence. Meeting and collaboration modules also include Agora, Socket.IO, and WebRTC-related code. Exact runtime topology depends on the deployment configuration and enabled meeting transport.
