---
title: Playwright and Jest-Playwright
categories:
- development
- ecmascript
- playwright
- playwright-jest
---
A Long Time Ago, In A Land Far, Far Away (California), an Empire (Google) created an e-2-e testing library called [Puppeteer](https://developers.google.com/web/tools/puppeteer), and it was...OK. Many people used it, but the powers-that-be kept pulling the strings, limiting it to working against Chrome (or, more specifically, a bundled version of Chromium).

The developers of Puppeteer felt oppressed. They knew they could do more, if only given the freedom to do so. They knew that they could help all developers. So they left the Death Star (still Google), and joined The Resistance (Microsoft? Really?) The Resistance was amassing a large array of fighters (Open Source Developers) to build and create new weapons (Open Source Software. Microsoft? Yeah, Microsoft) to take a stand against "the norm." This particular band of fighters then created [Playwright](https://playwright.dev/).

Playwright is quickly becoming the new end-to-end testing hotness. Unchained from the constraints forced upon them from their former employer, the Playwright team built a platform that helps all web developers. Operating agains all modern browsers: Chromium, Firefox and WebKit (even they skipped IE), Playwright provides a true cross-browser testing framework.

## Criteria
I decided to take a quick deep dive into Playwright. I began by defining a few criteria:
- I wanted to write tests in a syntax my team already understood
- I wanted the ability to have the tests login once, and propagate credentials across all tests
- I wanted to control my tests via a configuration that my CI environment could manipulate, allowing me to test against multiple tiers (local, Dev, QA, Pre-Production and Production)

Of course I started with Playwright's [Getting Started](https://playwright.dev/docs/intro). I created a repo specifically for keeping all of my tests, separating them entirely from my application code. This is non-standard, as most keep they're end-to-end tests side by side with their unit tests and their application code, but mine is a large enterprise app stitching together multiple frameworks and spanning a few dozen packages. Keeping it separate made more sense for me.

## Step 1: Install
My repository took some time to setup, and includes some scripts for it's own maintenance. For brevity, plus my sanity and yours, I'll give you a small subsection of the critical `devDependencies` required to meet my criteria.
```json
"devDependencies": {
  "@babel/core": "7.14.0",
  "@babel/plugin-transform-runtime": "7.13.15",
  "@babel/preset-env": "7.14.1",
  "babel-jest": "26.6.3",
  "config": "3.3.6",
  "core-js": "3.11.2",
  "eslint": "7.25.0",
  "eslint-plugin-jest-playwright": "0.3.2",
  "expect-playwright": "0.3.4",
  "jest": "26.6.3",
  "jest-playwright-preset": "1.5.2",
  "playwright": "1.10.0",
  "playwright-testing-library": "2.7.2",
  "prettier": "2.2.1",
  "regenerator-runtime": "0.13.7"
},
```
I'll also include the `eslint`, `babel` and `prettier` configurations from my `package.json` file.
```json
"eslintConfig": {
  "extends": [
    "plugin:jest-playwright/recommended"
  ],
  "plugins": [
    "jest",
    "jest-playwright"
  ],
  "env": {
    "node": true,
    "browser": true,
    "es6": true,
    "jest": true
  },
  "parserOptions": {
    "ecmaVersion": 2020,
    "sourceType": "module"
  },
  "rules": {
    "jest-playwright/missing-playwright-await": [
      "error",
      {
        "customMatchers": [
          "toHaveAttribute"
        ]
      }
    ],
    "block-scoped-var": 2,
    "camelcase": 0,
    "comma-style": [
      2,
      "last"
    ],
    "complexity": [
      2,
      {
        "max": 9
      }
    ],
    "curly": [
      2,
      "all"
    ],
    "dot-notation": [
      2,
      {
        "allowKeywords": true
      }
    ],
    "eqeqeq": [
      2,
      "always",
      {
        "null": "ignore"
      }
    ],
    "guard-for-in": 2,
    "max-depth": [
      2,
      {
        "max": 2
      }
    ],
    "max-len": [
      2,
      {
        "code": 100,
        "ignoreComments": true,
        "ignoreStrings": true,
        "ignoreTemplateLiterals": true,
        "ignoreRegExpLiterals": true
      }
    ],
    "max-params": [
      2,
      {
        "max": 10
      }
    ],
    "new-cap": [
      2,
      {
        "properties": false
      }
    ],
    "no-bitwise": 2,
    "no-caller": 2,
    "no-cond-assign": [
      2,
      "except-parens"
    ],
    "no-console": 1,
    "no-debugger": 2,
    "no-empty": 2,
    "no-eval": 2,
    "no-extend-native": 2,
    "no-irregular-whitespace": 2,
    "no-iterator": 2,
    "no-loop-func": 2,
    "no-multi-str": 2,
    "no-proto": 2,
    "no-script-url": 2,
    "no-sequences": 2,
    "no-shadow": 1,
    "no-undef": 2,
    "no-unused-vars": 2,
    "no-with": 2,
    "semi": [
      2,
      "always"
    ],
    "strict": 2,
    "valid-typeof": 2,
    "wrap-iife": [
      2,
      "inside"
    ]
  }
},
"babel": {
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": "current"
        }
      }
    ]
  ],
  "plugins": [
    "@babel/plugin-transform-runtime"
  ]
},
"prettier": {
  "printWidth": 80,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false
}
```
If you look through those configurations you'll see that the repo is setup for a `node` target (since Node runs the testing), and that the `eslint` configuration contains bits for working with the [jest-playwright-preset](https://github.com/playwright-community/jest-playwright). Also, I'm writing all of my tests in modern EcmaScript.

The last bit, for this, is to include some `scripts` in your `package.json`.

```json
"scripts": {
  "test": "jest -c jest.config.js",
  "lint": "eslint ./"
},
```

## Configuring for jest-playwright-preset
But, configurations don't stop there. We have to configure both `jest` and `jest-playwright-preset`, each through their own config files. And, since they're Node apps, they currently require the `require` and `module.exports` syntax.

jest.config.js
```js
module.exports = {
  preset: 'jest-playwright-preset',
  globalSetup: './__setups__/global-setup.js', // we'll get to it
  globalTeardown: './__setups__/global-teardown.js', // real soon
  testEnvironment: './__setups__/CustomEnvironment.js', // I promise
  testMatch: ['**/__tests__/**/*.js?(x)', '**/?(*.)+(spec|test).js?(x)'],
  transform: {
    '^.+\\.jsx?$': 'babel-jest',
  },
  setupFilesAfterEnv: ['./__setups__/regenerator.js', 'expect-playwright'], // I'll cover this too
};
```
OK, we're starting to get somewhere. Slightly. I worked all of this out over many hours, so I'm trying to make it simple for you. The `jest-playwright-preset` has a lot of things you can do in your **setup** and **teardown** phases for pre-configuring your test environment, and cleaning up when it's done. You'll see above that those files live in my `__setups__` folder, at my project root. We'll go over those a little more in a moment, but first let's add our next config:

jest-playwright.config.js
```js
const config = require('config'); // What's this? Just wait...

const {
  setup: { browsers, devices },
} = config;

module.exports = {
  // jest-playwright config
  browsers,
  exitOnPageError: false,
  launchOptions: {
    headless: true,
  },
  ...(devices && { devices }),
};
```
## And Then There's 'config'
So, [config](https://github.com/lorenwest/node-config) is a utility for configuring your Node applications. It allows you to define your application configuration using `json` and, through file naming convention, providing different configurations for the different environment tiers you're working with. Jest runs in a process environment of `TEST`, locally by default, so I place a `test.json` file in my root level `config` directory.

test.json
```json
{
  "protocol": "https://",
  "domain": "my.devdomain.com",
  "setup": {
    "browsers": ["chromium"],
    "organization": "myorg",
    "username": "foobar",
    "password": "F00B4r!"
  }
}
```
If you read line 1 of the `jest-playwright.config.js`, you'll see a `require` statement that gets my 'config'. Internally it does this by looking in your root level `config` directory for a json file name that matched the `process.ENV`. If it doesn't find a match, it automatically falls back to a `default.json` file. The above file is very basic, but you can get as intricate as you like for your tests. For quick testing of my persistent login I included dummy environment username and password in this file, but in practice you'll probably want those stored as 'secrets' in your CI environment.

## Don't forget the '\__setups__'
Let's talk about all of those 'other' files I referenced in the `jest` config. First and foremost, I'm not really bundling this code, so I need the necessary `core-js` polyfills and regenerator runtime available in my tests. This is easily handled:

`./__setups__/regenerator.js`
```js
import "core-js/stable";
import "regenerator-runtime/runtime";
```
Next I need a 'setup' that runs prior to all of my tests running. This is where I define my login process, and make my credentials persist.

`./__setups__/global-setup.js`
```js
import config from 'config';
import { globalSetup as playwrightGlobalSetup } from 'jest-playwright-preset';
import { chromium } from 'playwright';
// a method I predefined for credential persistance
import putStorageInProcess from '../src/helpers/putStorageInProcess.function';
// Get my vars from my `./config/[environment].json` file
const {
  protocol,
  domain,
  setup: { organization, username, password },
} = config;
// I pulled this into it's own function for clarity
async function doLogin(page) {
  // go to home page
  await page.goto(`${protocol}${domain}`);
  // fill in my username and password
  await page.fill('#username', username);
  await page.fill('input[type="password"]', password);
  // click the 'Sign In' button and wait for the redirect
  await Promise.all([page.waitForNavigation(), page.click('"Sign In"')]);
}

module.exports = async function globalSetup(globalConfig) {
  // get the jest-playwright stuff
  await playwrightGlobalSetup(globalConfig);

  // And all this stuff below is just standard Playwright stuff
  const browser = await chromium.launch(); // get a 'browser' instance
  const context = await browser.newContext(); // setup a new browser 'context'
  const page = await context.newPage(); // get a 'page' from the 'context'

  await doLogin(page); // do my login

  await putStorageInProcess(page, context); // persist my credentials
  // close the context and the browser, to prevent memory leak
  await context.close();
  await browser.close();
};
```
Now, I abstracted the credential storage process into it's own reusable method, because occasionally I'll need to call it again during various tests. For my application, our login places bit in `localStorage`, `sessionStorage` and in `cookies`. But Playwright only maintains that info for the life of a single test, so I have to reset it all before each test. For now, we'll look at how you store your credentials for reuse.

`./src/helpers/putStoraginInProcess.js`
```js
export default async function putStorageInProcess(page, context) {
  // first I'll get my 'localStorage'
  const storage = await context.storageState();
  process.env.STORAGE = JSON.stringify(storage); // and set it on an Environment variable
  // next I'll get my 'sessionStorage'
  const session = await page.evaluate(() =>
    JSON.stringify(window.sessionStorage)
  );
  process.env.SESSION_STORAGE = session; // and set it on an Environment variable
  // last I'll get my 'cookies'
  const cookies = await context.cookies();
  process.env.COOKIES = JSON.stringify(cookies); // and set them on an Environment variable
}
```
It's entirely possible, since we're using Node, to just write these storage things to text files in the system. But, my app persistence footprint is pretty small, and file I/O is costly, so I just put them in Environment variables through testing.

Next I define my teardown process, that runs only after all test suites have completed.

`./__setups__/global-teardown.js`
```js
import { globalTeardown as playwrightGlobalTeardown } from 'jest-playwright-preset';

module.exports = async function globalTeardown(globalConfig) {
  // Your global teardown
  await playwrightGlobalTeardown(globalConfig);
};
```
The `jest-playwright-preset` allows you to setup custom test environments. You can do a variety of things this way, but my example is taken right off their documentation for taking screenshots during test failures:

`./__setups__/CustomEnvironment.js`
```js
const PlaywrightEnvironment = require('jest-playwright-preset/lib/PlaywrightEnvironment')
  .default;

class CustomEnvironment extends PlaywrightEnvironment {
  async setup() {
    await super.setup();
    // Your setup
  }

  async teardown() {
    // Your teardown
    await super.teardown();
  }

  async handleTestEvent(event) {
    if (event.name === 'test_done' && event.test.errors.length > 0) {
      const parentName = event.test.parent.name.replace(/\W/g, '-');
      const specName = event.test.name.replace(/\W/g, '-');

      await this.global.page.screenshot({
        path: `./screenshots/${parentName}_${specName}.png`,
      });
    }
  }
}

module.exports = CustomEnvironment;
```
And, we're finally finished with handling our Playwright testing configuration with `jest-playwright-preset`. All that's left is to write a test!

## Writing a Test

OK, I lied. I'm almost done with configuration. Our `global-setup` put our login credentials away for later, and now it's later. To understand why, let's look at an initial test suite without some stubbs where tests go.

`./src/AppArea.spec.js`
```js
import config from 'config';
import { setupBefore, setupAfter } from './beforeAndAfter';

const { protocol, domain } = config;

jest.setTimeout(35 * 1000); // give jest-playwright a little more time than jest's standard timeout

describe('Some Area of My Application', () => {
  /**
   * This will
   * - setup our session prior to each test, based on the login in globalSetup
   * - pull and store all updated session state in environment vars after each test
   */
  beforeEach(setupBefore);
  afterEach(setupAfter);

  test('should do something', async () => {
    // your test goes here
  });

  test('should do something else', async () => {
    // your next test goes here
  });
});
```
This is boilerplate stuff. It'll typically be identical for every test suite you write. As it says in the comments, we want to recreate our user credentials before each test, and save off any changes after every test. I have an `index.js` file in that `beforeAndAfter` directory that imports those functions from there respective files, then exports them from one centralized location.

`./src/beforeAndAfter/setupBefore.function.js`
```js
import config from 'config'; // get variables

export default async () => {
  const storageState = JSON.parse(process.env.STORAGE); // parse the stored 'localStorage'
  await jestPlaywright.resetContext({ storageState }); // and apply it to the 'context'
  const sessionStorage = process.env.SESSION_STORAGE;
  // This block is the hoops we jump through to apply our 'sessionStorage'
  await context.addInitScript((storage) => {
    if (window.location.hostname === config.domain) {
      const entries = JSON.parse(storage);
      Object.keys(entries).forEach((key) => {
        window.sessionStorage.setItem(key, entries[key]);
      });
    }
  }, sessionStorage);
  const deserializedCookies = JSON.parse(process.env.COOKIES); // parse the stored 'cookies'
  await context.addCookies(deserializedCookies); // and apply them to the 'context'

  await jestPlaywright.resetPage({}); // and this gives us a fresh 'page' for each test
};
```
And **NOW** you're ready to start writing your tests. I'm not going to include any here, but I'll give you a few things to know ahead of time.
- `jest-playwright` has some 'global' variables that are available with each test. Unlike our `global-setup.js`, which runs while `jest-playwright` is setting up, you do not have to manually instantiate these instances. They are automatically available. You should really read the Documentation.
  - `browserName` - which browser a test is running under
  - `deviceName` - which device the test is currently running as
  - `browser` - the current jest-playwright [Browser](https://playwright.dev/docs/api/class-browser/) instance
  - `context` - the current jest-playwright [Context](https://playwright.dev/docs/api/class-browsercontext/) instance for each new test suite/file
  - `page` - the current jest-playwright [Page](https://playwright.dev/docs/api/class-page/) instance for each test
- As the `jest-playwright.config.js` showed you, you setup the `browsers` and `devices` you wish to test against in your json config file. Each test will be run against each browser and device you define. You should really read the Documentation.
- **ALL** Playwright/jest-playwright/[expect-playwright](https://github.com/playwright-community/expect-playwright) methods are asynchronous, so your tests will use `async/await`. You should really read the Documentation.
- Each test begins with you going to a `page`. You should really read the Documentation.

Can you tell I'm a fan of good documentation?

Alright, I'll give you one test. Let's say I want to test... my login page. I don't really have to do this, since my `global-setup.js` is already doing it for me, but let's do it anyway.

`./src/Login.spec.js`
```js
import config from 'config';
// We're skipping this for this test
// import { setupBefore, setupAfter } from './beforeAndAfter';

const { protocol, domain } = config;

jest.setTimeout(35 * 1000); // give jest-playwright a little more time than jest's standard timeout

describe('Some Area of My Application', () => {
  // And we won't need these, as it'll just mess up the login
  //beforeEach(setupBefore);
  //afterEach(setupAfter);

  test('should log us in', async () => {
    // go to our domain
    await page.goto(`${protocol}${domain}`);
    // login
    await page.fill('input[type="text"]', username);
    await page.fill('input[type="password"]', password);
    // click the 'Sign In' button and wait for nav transition
    await Promise.all([page.waitForNavigation(), page.click('"Sign In"')]);
    // check to see that we logged in by finding the H1 saying 'Home Page'
    await expect(page).toHaveSelector('"Home Page"');
  });
});
```

Playwright and `jest-playwright-preset`, in conjunction with tools like `expect-playwright` and [playwright-testing-library](https://github.com/hoverinc/playwright-testing-library), have a lot of power and functionality to let you write true end-to-end-testing. Documentation is extensive, and I suggest taking a good day (or five) to familiarize yourself with all of the capabilities they provide.