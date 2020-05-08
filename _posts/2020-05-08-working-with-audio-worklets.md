---
title: SoundTouchJS
categories:
- development
- ecmascript
- webaudioapi
- soundtouchjs
- audioworklets
---
As I mentioned, in a [previous post](https://cutterscrossing.com/development/ecmascript/webaudioapi/2020/04/01/soundtouchjs/), I'm the steward of the open source [SoundTouchJS](https://github.com/cutterbl/SoundTouchJS) audio processing library. I did not start `SoundTouchJS`, merely reviving a largely dead project, modernizing the code, and providing some core updates. I am not an expert of the [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) but, as a singer and a developer, I've always been interested in audio processing in the browser. I've played around with `SoundTouchJS` for a few years now, but one thing I always wanted to do, but never had the motivation, was create an [AudioWorklet](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorklet) implementation. Luckily [Janick Delot](https://github.com/watch-janick) reached out to me about sponsoring this work for `SoundTouchJS`.

## Why?

For those who are unfamiliar, some time ago the [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) was introduced. Conceptually the idea is simple; sometimes we need to run process intensive code for our applications. Running that code, in the main browser context thread, can create performance bottlenecks and slow down rendering and function. Using Web Workers we can offload some of this process intensive code to separate processing contexts, outside of the main context, thereby reducing the bottlenecks and improving overall application performance.

## Enter the AudioWorklet

`AudioWorklets` are a targeted type of `Web Worker`. They allow you to offload process intensive audio processing to their own separate processing contexts. Implementing an `AudioWorklet` requires two, separate objects (defined by the `Web Audio API`); an [AudioWorkletProcessor](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorkletProcessor) and an [AudioWorkletNode](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorkletNode). The `AudioWorkletProcessor` *is* the processing script that works as it's own Web Worker context, while the `AudioWorkletNode` is the communications hub between the processor script and the [AudioContext](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext).

## Okay....

Confused yet? So was I. There were several challenges to overcome in creating an implementation. Like "How do I load the processor script to the browser?" There's a tiny bit of Black Magic involved in this process, ensuring that your web server serves the file with the proper content header (this is true of all `Web Worker`s). There were challenges in understanding the differences between how the `AudioWorkletProcessor` handles the buffer [process](https://developer.mozilla.org/en-US/docs/Web/API/AudioWorkletProcessor/process), in comparison to that of the deprecated [ScriptProcessor audioprocess event](https://developer.mozilla.org/en-US/docs/Web/API/ScriptProcessorNode/audioprocess_event). There were also challenges with `SoundTouchJS` itself, which currently requires access to the entire [AudioBuffer](https://developer.mozilla.org/en-US/docs/Web/API/AudioBuffer) in advance, in handling the stretch and rate transposition duties it performs. And then there's the real kicker; browser support. While the `AudioWorklet`s work swimmingly in all Chromium based browsers (Chrome, Edge, Electron...) and Firefox (somewhat), there is no support in Safari. This required testing with a ponyfill like [standardized-audio-context](https://github.com/chrisguttandin/standardized-audio-context), that normalized the API across all browsers and created fallbacks where features were unsupported. In Safari, for instance, SAC would take the `AudioWorkletProcessor` and convert it to the deprecated [ScriptProcessorNode](https://developer.mozilla.org/en-US/docs/Web/API/ScriptProcessorNode) automagically (which, by the way, runs in the main browser context).

## Success!!!

In the end, after burning quite a bit of midnight oil and several incantations to various deities of multiple religions (as well as some choice words that shouldn't be repeated around women and children) we now have the [@soundtouchjs/audio-worklet](https://github.com/cutterbl/soundtouchjs-audio-worklet), an `AudioWorklet` for implementing `SoundTouchJS` in it's own processing context (where supported). **Thanks to Janick**, again, for supporting open source development. Also a **huge Thank You** to [Christoph Guttandin](https://github.com/chrisguttandin), not only for his work on `standardized-audio-context`, but also hammering through some implementation details with me and providing some deeper insight into the `Web Audio API` itself.

## Final Thoughts

My one true downer, throughout this process, has been [Apple](https://apple.com). I've been an Apple convert for some time. I love their devices, but I never use Safari, and this sorta nailed that one. The `Web Audio API` spec has been around for a while, and yet it seems that Apple has no interest in following it. They still implement via their [webkitAudioContext](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Migrating_from_webkitAudioContext), instead of the standard `AudioContext` (and with no documentation I can find), and it still solely implements features like the `ScriptProcessorNode` (which was deprecated by the spec in August of 2014), while not supporting current specifications like the `AudioWorklet`. I'm sure there's some strange reasoning, behind their lack of support, but it really feels as if they're just ignoring audio in the browser to me.

On the other hand, as someone who writes Occupational Health software for a living, it's been a lot of fun to focus on something other than the Corona Virus during off hours.