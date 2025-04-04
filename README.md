# [Transformers.js](https://huggingface.co/docs/transformers.js/en/index) + NuxtJS

Here's a basic demo using these two tools. Transformers.js is typically used within a web worker for several important performance reasons:

-   Main thread protection
-   Computation isolation
-   Memory management
-   Parallelization

This is a barebones demo showing Transformers.js working with Whisper in the browser for realtime audio to text transcription.
It uses [whisper-base](https://huggingface.co/onnx-community/whisper-base), a 290mb model, and is able to transcribe audio in all of the languages listed [here](https://github.com/openai/whisper/blob/248b6cb124225dd263bb9bd32d060b6517e067f8/whisper/tokenizer.py#L79). It can be ran entirely offline once the model is loaded in.

![realtime transcription showcase](https://i.imgur.com/RutByr0.gif)

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

## Not working?

You may be running into the a webgpu error. Check your browser's console to see if "Uncaught (in promise) Error: no available backend found. ERR: [webgpu] Error: Failed to get GPU adapter. You may need to enable flag "--enable-unsafe-webgpu" if you are using Chrome." is showing, do the following:

### Enable the WebGPU Flag:
1. Open Chrome and navigate to chrome://flags in the address bar.
2. Search for #enable-unsafe-webgpu.
3. Set the flag to Enabled.
4. Relaunch Chrome when prompted.
