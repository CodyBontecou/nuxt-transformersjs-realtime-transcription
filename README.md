# [Nuxt Workers](https://github.com/danielroe/nuxt-workers) + [Transformers.js](https://huggingface.co/docs/transformers.js/en/index)

Here's a basic demo using these two tools. Transformers.js is typically used within a web worker for several important performance reasons:

- Main thread protection
- Computation isolation
- Memory management
- Parallelization

This is a barebones demo showing Transformers.js working within a Nuxt application alongside Daniel Roe's worker module.

This code is deployed here - https://nuxt-workers-transformersjs.vercel.app/

**Note:** The model [Xenova/nllb-200-distilled-600M](https://huggingface.co/Xenova/nllb-200-distilled-600M) will begin installing on page load. It will take a bit of time depending on your internet speed.
