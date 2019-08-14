# react-native-typed-postcss-transformer

[![NPM version](http://img.shields.io/npm/v/react-native-typed-postcss-transformer.svg)](https://www.npmjs.org/package/react-native-typed-postcss-transformer)
[![Downloads per month](https://img.shields.io/npm/dm/react-native-typed-postcss-transformer.svg)](http://npmcharts.com/compare/react-native-typed-postcss-transformer?periodLength=30)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://egghead.io/courses/how-to-contribute-to-an-open-source-project-on-github)

Use [PostCSS](https://github.com/postcss/postcss) to transform CSS to [react native style objects](https://facebook.github.io/react-native/docs/style.html).

This transformer also generates `.d.ts` Typescript typings for the CSS files. Notice that platform specific extensions are not supported in the Typescript typings.

> This transformer can be used together with [React Native CSS modules](https://github.com/kristerkari/react-native-css-modules).

_Minimum React Native version for this transformer is 0.52. If you are using an older version, please update to a newer React Native version before trying to use this transformer._

## Usage

### Step 1: Install

```sh
yarn add --dev react-native-typed-postcss-transformer postcss
```

### Step 2: Add your PostCSS config and install your PostCSS plugins

Add your PostCSS configuration to [one of the supported config formats](https://github.com/michael-ciniawsky/postcss-load-config), e.g. `package.json`, `.postcssrc`, `postcss.config.js`, etc.

### Step 3: Configure the react native packager

#### For React Native v0.57 or newer / Expo SDK v31.0.0 or newer

Add this to `metro.config.js` in your project's root (create the file if it does not exist already):

```js
const { getDefaultConfig } = require("metro-config");

module.exports = (async () => {
  const {
    resolver: { sourceExts }
  } = await getDefaultConfig();
  return {
    transformer: {
      babelTransformerPath: require.resolve("./postcss-transformer.js")
    },
    resolver: {
      sourceExts: [...sourceExts, "css", "pcss"]
    }
  };
})();
```

If you are using [Expo](https://expo.io/), you also need to add this to `app.json`:

```json
{
  "expo": {
    "packagerOpts": {
      "config": "metro.config.js",
      "sourceExts": ["ts", "tsx", "css", "pcss"]
    }
  }
}
```

---

#### For React Native v0.56 or older

If you are using React Native without Expo, add this to `rn-cli.config.js` in your project's root (create the file if you don't have one already):

```js
module.exports = {
  getTransformModulePath() {
    return require.resolve("./postcss-transformer.js");
  },
  getSourceExts() {
    return ["ts", "tsx", "css", "pcss"]; // <-- Add other extensions if needed.
  }
};
```

---

#### For Expo SDK v30.0.0 or older

If you are using [Expo](https://expo.io/), instead of adding the `rn-cli.config.js` file, you need to add this to `app.json`:

```json
{
  "expo": {
    "packagerOpts": {
      "sourceExts": ["ts", "tsx", "css", "pcss"],
      "transformer": "./postcss-transformer.js"
    }
  }
}
```

### Step 4: Add transformer file

Create `postcss-transformer.js` file to your project's root and specify supported extensions:

```js
// For React Native version 0.59 or later
var upstreamTransformer = require("metro-react-native-babel-transformer");

// For React Native version 0.56-0.58
// var upstreamTransformer = require("metro/src/reactNativeTransformer");

// For React Native version 0.52-0.55
// var upstreamTransformer = require("metro/src/transformer");

// For React Native version 0.47-0.51
// var upstreamTransformer = require("metro-bundler/src/transformer");

// For React Native version 0.46
// var upstreamTransformer = require("metro-bundler/build/transformer");

var postcssTransformer = require("react-native-typed-postcss-transformer");
var postCSSExtensions = ["css", "pcss"]; // <-- Add other extensions if needed.

module.exports.transform = function({ src, filename, options }) {
  if (postCSSExtensions.some(ext => filename.endsWith("." + ext))) {
    return postcssTransformer.transform({ src, filename, options });
  }
  return upstreamTransformer.transform({ src, filename, options });
};
```

## How does it work?

Your `App.css` file might look like this (using [postcss-css-variables](https://github.com/MadLittleMods/postcss-css-variables) plugin):

```css
:root {
  --blue-color: blue;
}

.myClass {
  color: var(--blue-color);
}
.myOtherClass {
  color: red;
}
```

When you import your stylesheet:

```js
import styles from "./App.css";
```

Your imported styles will look like this:

```js
var styles = {
  myClass: {
    color: "blue"
  },
  myOtherClass: {
    color: "red"
  }
};
```

The generated `App.css.d.ts` file looks like this:

```ts
export const myClass: string;
export const myOtherClass: string;
```

You can then use that style object with an element:

```jsx
<MyElement style={styles.myClass} />
```

## TODO

- Find a way to make the configuration cleaner by only having to add `rn-cli.config.js` to a project, https://github.com/kristerkari/react-native-postcss-transformer/issues/1.
