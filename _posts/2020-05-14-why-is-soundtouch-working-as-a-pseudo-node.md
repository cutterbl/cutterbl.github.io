---
title: "Why Is SoundTouch Working As A Pseudo Node?"
categories:
- development
- ecmascript
- webaudioapi
- soundtouchjs
- audioworklets
---
GitHub user [@floydback](https://github.com/floydback), in an Issue in the [@soundtouchjs/audio-worklet](https://github.com/cutterbl/soundtouchjs-audio-worklet) project, asked the following question:

> Do you understand why SoundTouch can't work like a real node? Is there a fundamental problem? I mean AudioNode like GainNode or DelayNode. Now you need to connect/disconnect to the destination each time when play/stop audio. Is there a fundamental reason why PitchShifter cannot be connected to? Another drawback you can't use it with MediaElementSourceNode without decoding audio data.

You want to know why the SoundTouchNode is basically always the first node in the chain (after the AudioBufferNode)? And why it doesn't read the input stream, but the actual AudioBuffer. This is laid out a little better in the parent [SoundTouchJS](https://github.com/cutterbl/SoundTouchJS) project, but let me explain it a little deeper.

I did not write SoundTouchJS. If you track the [In Case You Are Interested](https://github.com/cutterbl/SoundTouchJS#in-case-you-are-interested) section of the README you'll see that the project [started back in 2011](https://github.com/also/soundtouch-js), and quickly died. The PitchShifter came from a [fork back in 2015](https://github.com/jakubfiala/soundtouch-js), and again development stopped.

I found all of this out-of-date code back in 2018, when I needed a Web Audio API "key changer" for some [Karaoke software](https://github.com/cutterbl/CDGPlayer) I was writing. I cobbled all of these bits together, refactoring the code into modern ES6 classes, and adding a few minor tweaks along the way, so that I, or anyone else, could import the classes into my projects.

And, while I've read all of this code, I *do not* claim to understand all of it. I'm a UI/UX engineer, not an expert in the Web Audio API. Music is my passion, not my job. There is some crazy sound-science math going on in there for handling the pitch shifting. What I can tell you is that it does this:

* Takes the entire AudioBuffer, up-front
* extracts multiple sample frames
* after a specific limit of frames is added to it's internal `input buffer`, it
* performs stretch and rate transposition logic, to match the required pitch levels the user has requested, through an internal `intermediary buffer` to an internal `output buffer`
* then it dumps that internal `output buffer` to the node output stream

After all of my work, creating the new AudioWorklet, I've gained a higher understanding of some of the Web Audio API bits. It would, absolutely, be better if the 'Node' would read from the input stream, rather than the AudioBuffer. I've been digging in to the SoundTouchJS internals, really tracking what it 'does', and have found that, while it does what it is supposed to do from an audio perspective, it isn't the most performant code. I am already stepping through some things to attempt to read from the Node's input stream, rather than requiring the AudioBuffer up-front.

And so, I move forward. Slowly. Cautiously. Because, while I want to make all of this better, and available to everyone, I'm basically a one man band. And this isn't my job, it's a side deal, open source project. And, ultimately, I want it to work (which it does), so changes have to go slow. Other people use this project, and we don't want their code to break without warning.

So, if you're out there, and you're reading this, and you're so inclined, [SoundTouchJS](https://github.com/cutterbl/soundTouchJS) is an Open Source project. We do welcome pull requests.