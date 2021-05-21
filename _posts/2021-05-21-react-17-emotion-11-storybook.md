---
title: React 17 + Emotion 11 + Storybook 6
categories:
- development
- ecmascript
- react
- emotion
- storybook
---
So I was attempting to update my [@cxing/date-selector](https://cutterscrossing.com/date-selector) project today. Updating dependencies can be a chore, sometimes, and this was no exception.

**Important Note: This is not a Create React App (CRA) project. The steps listed here will not work for those.**

First, let me give you a rundown of the major dependency upgrades:
- Babel
- React
- Emotion

I've done upgrades to React 17 before, but I had not done so using the [new JSX Transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html). Here's some step by step:

1. Update `react` and `react-dom`
2. Update `@babel/core`, `@babel/preset-env`, `@babel/preset-react`, and any other 'babel' dependencies
3. Setup for the new JSX Transform
```js
// from babel.config.js
    presets: [
      '@babel/preset-env',
      [
        '@babel/preset-react',
        { runtime: 'automatic' },
      ],
    ],
```
*Side Note: Babel docs say that starting with Babel 8 "automatic" will become the default.*
4. Configure eslint for `eslint-plugin-react`
```js
// from "eslingConfig" in package.json
    "rules": {
      "react/jsx-uses-react": "off",
      "react/react-in-jsx-scope": "off"
    }
```
5. Remove unused React imports
```
# in your terminal
npx react-codemod update-react-imports
```

So, that takes care of the basics of the React 17 upgrade, but now I have to handle Emotion 11. You see, Emotion has it's own JSX transformer, and requires a bit more work to get it going.

1. Update from older versions by installing new dependencies
```
@emotion/babel-plugin
@emotion/eslint-plugin
@emotion/react
```
2. Update your code, by removing the old JSX Pragma declarations
```js
// replace all of these

/** @jsx jsx */
import { jsx } from '@emotion/core';
// with this

// eslint-disable-next-line no-unused-vars
import { jsx } from '@emotion/react';
```
3. Then update your `babel.config.js` to use Emotion's transformer
```js
// from babel.config.js
    presets: [
      '@babel/preset-env',
      [
        '@babel/preset-react',
        { runtime: 'automatic', importSource: '@emotion/react' },
      ],
    ],
```
4. But, you'll also need to add the plugin to that config
```js
    plugins: ['@emotion/babel-plugin'],
```
5. Update your eslint configuration
```
// from your "eslintConfig" in your package.json
    "rules": {
      "@emotion/pkg-renaming": "error",
      "@emotion/jsx-import": "off",
      "@emotion/no-vanilla": "error",
      "@emotion/import-from-emotion": "error",
      "@emotion/styled-import": "error",
      // ... other rules
    }
```
This should be enough to build your project again, hopefully without any other code changes. But, this is where I ran into trouble. I use [Storybook](https://storybook.js.org) to develop in isolation, and document my open source works. First thing I did, after getting a working build, was fire up Storybook to make sure everything worked.

And, it didn't.

**This is the part that isn't in the Emotion documentation.** After quite a while digging, I found that Storybook's internal Webpack configuration was not using my root level `babel.config.js`. So, it was missing some key things to work properly. Luckily, a quick update to my `./.storybook/main.js` took care of my issue.
```js
// ./.storybook/main.js
module.exports = {
  stories: ['../src/**/*.stories.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: ['@storybook/addon-links', '@storybook/addon-essentials'],
  webpackFinal: async (config) => {
    config.module.rules[0].use[0].options.presets = [
      require.resolve('@babel/preset-env'),
      [
        require.resolve('@babel/preset-react'),
        {
          runtime: 'automatic',
          importSource: '@emotion/react',
        },
      ],
    ];

    config.module.rules[0].use[0].options.plugins = [
      ...config.module.rules[0].use[0].options.plugins,
      '@emotion/babel-plugin',
    ];

    return config;
  },
};
```

Voila! Restarting Storybook recompiled as it should, and I was able to see my styling again.

It seems so easy, when I look at it now, but actually took up some time to figure everything out. Hopefully this will help other's avoid the pain I went through to get this upgrade working properly. 