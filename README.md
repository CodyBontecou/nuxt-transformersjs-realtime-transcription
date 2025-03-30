# [Transformers.js](https://huggingface.co/docs/transformers.js/en/index) + NuxtJS

![realtime transcription showcase](https://i.imgur.com/RutByr0.gif)

Here's a basic demo using these two tools. Transformers.js is typically used within a web worker for several important performance reasons:

-   Main thread protection
-   Computation isolation
-   Memory management
-   Parallelization

This is a barebones demo showing Transformers.js working with Whisper in the browser for realtime audio to text transcription.

This code is deployed here - https://nuxt-transformersjs-realtime-transcription.vercel.app/


## Run locally

- Install dependencies.

```bash
npm install
```

- Run local server.

```bash
npm run dev
```

- Navigate to local server. The default is localhost:3000.
