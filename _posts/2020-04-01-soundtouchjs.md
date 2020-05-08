---
title: SoundTouchJS
categories:
- development
- ecmascript
- webaudioapi
- soundtouchjs
---
Some time back I became obsessed with writing my own browser-based Karaoke software. Those who know me know I love to sing. It's how I met my wife (a former KJ). It's how we got married (on a Karaoke stage). It's [weekend fun time](https://www.youtube.com/channel/UC42Nrsga5KEa9Wy3ftLnZ8g) with our kid.

But, you can't do Karaoke without being able to change the key the song is in. Well, you can, but I can't sing Queen in the same key as Freddy Mercury anymore. So, I went searching for JS libraries for doing this.

What I found was mostly bupkus. I did, eventually, stumble on some old work to create a JS version of the C++ [SoundTouch](https://www.surina.net/soundtouch/) library. This led me to some forks, where I found a PitchShifter example, and then I just had to dig in.

I rewrote all of it ground up, exporting everything as ES2015 classes and functions. I've had it out there for awhile on [GitHub](https://github.com/cutterbl/SoundTouchJS) and [NPM](https://www.npmjs.com/package/soundtouchjs). I thought it was about time I let other's know that it's available.

Although I have a straight ES2015 example in the repo, I recently created a React example, consuming and using the library.

<iframe
     src="https://codesandbox.io/embed/soundtouchjs-with-react-qdci0?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="SoundTouchJS with React"
     allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
     sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
   ></iframe>