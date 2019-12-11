- Feature Name: `tooling_javascript_linting_formatting`
- Start Date: 2019-10-30
- RFC PR: [signal-noise/rfcs#0004](https://github.com/signal-noise/rfcs/pull/0004)

# Summary
[summary]: #summary

We try to ensure consistent code format and linting across all our javascript projects. We accomplish this using four tools:

- The industry standard [ESLint](https://eslint.org/) for JavaScript linting
- [StyleLint](https://stylelint.io/) for linting styles
- [Prettier](https://github.com/prettier/prettier) for code formatting
- [Import Sort](https://github.com/renke/import-sort) for more standardised import declarations

# Motivation
[motivation]: #motivation

Over the course of a developer's career they pick up their own code style, be it tabs or spaces, single or double quotes or the many other stylistic choices they make.

The suggested approach to address this has many benefits including...

- Eliminates some of the on-going debates over styles and asks developers to all code in a consistent way.
- Reduces mental load switching between projects.
- Reduces conflicts when merging branches.

As a general rule, we use the most commonly-accepted and default configuration of each of these tools. The exception is when a different configuration creates a more logical system (as opposed to simply being personal preference).

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

To utilise the standard tooling there are two sides. First being the project setup and secondly setting up your IDE to support these tools. For an explanation of which tools we use and why please see there [Reference-level explanation](#reference-level-explanation)

## Project setup
[project-setup]: #project-setup

A reference project setup can be found in the [tooling template](https://github.com/signal-noise/tooling_js), but this may be out of date.

To set all the tools up from scratch is also relatively easy, as follows.

### Installation

Install [ESLint](#eslint) to catch errors and stylistic problems in your code quickly.

`npm install eslint --save-dev`

Install [Prettier](#prettier) to keep your code formatted like the rest of the team's.

`npm install prettier eslint-plugin-prettier --save-dev`

Install [StyleLint](#stylelint) to keep your CSS clean and help catch errors early.

`npm install stylelint stylelint-order stylelint-config-recommended stylelint-config-prettier stylelint-config-rational-order --save-dev`

If you're using Styled Components you will also want to install `stylelint-processor-styled-components` and `stylelint-config-styled-components`, and you may not want to enforce ordering rules since they can't be automatically fixed in Styled Components.

If you're using a design system you should also install `@signal-noise/stylelint-scales`.

Install [import-sort](#import-sort) to enforce the order of your imports.

`npm install import-sort-cli import-sort-parser-babylon import-sort-style-module --save-dev`

### Configuration

We favour `.rc` files for project level configuration. Example configurations are as follows

#### `.eslintrc`

Basic linting setup for a modern React project.

```javascript
{
  "env": {
    "browser": true,
    "node": true,
    "es6": true,
    "jest": true
  },
  "extends": ["eslint:recommended", "plugin:react/recommended"],
  "plugins": ["react", "react-hooks"],
  "parser": "babel-eslint",
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}

```

#### `.stylelintrc`

Basic stylelint setup for a project using styled-components with some design system rules added.

```javascript
{
  // Enabled Parsing of styled-components
  "processors": ["stylelint-processor-styled-components"],

  "extends": [
    // Base built-in config
    "stylelint-config-recommended",
    "stylelint-config-prettier",

    // styled-components config
    "stylelint-config-styled-components",

    // Rational Order config
    "stylelint-config-rational-order"
  ],

  // Extra rules for ensuring design consistency
  "plugins": ["@signal-noise/stylelint-scales"],
  "rules": {
    "scales/font-family": [["IBM Plex Sans", "DecimaMono", "sans-serif"]],
    "scales/font-size": [[11, 14, 16, 24, 32, 48], { "unit": "px" }],
    "scales/font-weight": [[200, 300, 400, 700]],
    "scales/radii": [[]],
    "scales/space": [[4, 8, 16, 32, 64, 128], { "unit": "px" }],
    "scales/sizes": [[1, 16, 32, 64, 128, 256, 100, 800]],
    "scales/color": [
      [
        [0, 0, 0],
        [255, 255, 255],
        [205, 255, 0],
        [250, 106, 118],
        [70, 73, 178],
        [11, 12, 33]
      ]
    ],
    "scales/border-width": [[1], { "unit": "px" }],
    "scales/z-indices": [[0, 1, 2, 3, 4, 5, 6, 7, 8]]
  }
}
```

#### `.importsortrc`

Setup to configure our import sorting.

```javascript
{
  ".js, .jsx": {
    "parser": "babylon",
    "style": "module"
  }
}
```

### `package.json`

We also try to keep the scripts defined within `package.json` consistent between projects like so...

```javascript
{
  "scripts": {
    "lint": "npm-run-all --parallel lint:*",
    "lint:js": "eslint \"**/*.js\"",
    "lint:css": "stylelint '{src/theme,src/layouts,src/components}/**/*.{js,jsx}'",
    "lint:import": "import-sort -l 'src/**/*.{js,jsx}'",
    "lint:format": "prettier \"**/*.{js,html,yaml,yml}\" --check",
    "fix": "npm-run-all --sequential fix:*",
    "fix:js": "eslint \"**/*.js\" --fix",
    "fix:import": "import-sort --write 'src/**/*.{js,jsx}'",
    "fix:format": "prettier \"**/*.{js,html,yaml,yml}\" --write"
  }
}
```

## IDE support
[ide-support]: #ide-support

At Signal Noise most developers use [Visual Studio Code](https://code.visualstudio.com/) so below is a small list of plugins that can  highlight and help fix issues raised by the tooling:

- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [sort-imports](https://marketplace.visualstudio.com/items?itemName=amatiasq.sort-imports)
- [ES7 React/Redux/GraphQL/React-Native snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets)

> This setup can be accomplished in most common IDEs using similar plugins

Please note these tools (ESLint, Prettier etc) only pick up configuration when defined in the root of the folder opened in VS Code. So if you have your frontend code nested in a folder, e.g. `/www/src` you will need to open `/www` as the root folder. For further information please check this [issue](https://github.com/prettier/prettier-vscode/issues/371).

If you encounter any inconsistent behavior when running Prettier please read [this documentation](https://github.com/prettier/prettier-vscode#settings) to ensure you are not overwriting any defaults.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Our approach to tooling is split into various areas:

- [Formatting]
  - [Prettier]
  - [Import Sort](#import-sort)
- [Linting]
  - [ESLint]
  - [StyleLint]

## Formatting
[formatting]: #formatting

This covers the _code style_ domain of tooling.

### Prettier
[prettier]: #prettier

[Prettier](https://github.com/prettier/prettier) is an opinionated code formatter with support for the majority of file types we create day-to-day within a Javascript project including JavaScript itself, JSON, CSS, HTML and Markdown amongst others. The prettification of JavaScript files are required but other formats can be adopted on a per-project basis.

We do not have the prettification of JSON, Markdown etc as a rule as these are often computer generated and out of the developers control to maintain the format. CSS and HTML should ideally be linted if it is able to be fully automatic and causes no headaches for developers.

Prettier has many configuration options that allows the style to be customised but we adopt the zero-configuration approach as this marginally reduces setup time and further eliminates the debates mentioned above.

The core defaults with Prettier are:

- 80 character lint width
- 2 space tab width
- "Double" quotes
- Enforce use of semicolons
- No trailing commas

### Import Sort
[import-sort]: #import-sort

The order of imports/requires within files can be a bane of developers when it comes to merge conflicts. Without enforcement they can appear as seemingly random or first-imported order.

We use [import-sort](https://github.com/renke/import-sort) to enforce the order of our imports and also the [module](https://github.com/renke/import-sort/tree/master/packages/import-sort-style-module) order styling.

The `module` style ordering is like so:

1. Absolute modules with side effects, e.g.`import "React";`
2. Relative modules with side effects, e.g.`import "./customPolyfill";`
3. Standard node modules, e.g. `import {readFile} from "fs";`
4. Third-party modules, e.g. `import useApi from "@signal-noise/useApi";`
5. First-party modules, e.g. `import Footer from "../components/Footer";`

Imports at the same level are then sorted alphabetically.

## Linting
[linting]: #linting

This covers the _linting_ domain of tooling.

### ESLint
[eslint]: #eslint

[ESLint](https://eslint.org/) is an open source JavaScript linting utility. Code linting is a type of static analysis that is frequently used to find problematic patterns or code that doesn’t adhere to certain style guidelines.

We use a minimal configuration that extends the `eslint:recommended` preset. Other plugins can be included based on the type of project, for example `react/recommended` for React projects.

### StyleLint
[stylelint]: #stylelint

[StyleLint](https://stylelint.io/) is a modern CSS styles linter that helps you avoid errors and enforce conventions in your styles.

We adopt the standard `stylelint-config-recommended` along with `stylelint-config-rational-order` rules and add extra rules on a per-project basis like `stylelint-config-styled-components`.

We have also developed our own rule plugin ([@signal-noise/stylelint-scales](https://github.com/signal-noise/stylelint-scales)) that can help enforce strict design systems. This is design system approach is outlined in detail in another RFC.

# Drawbacks
[drawbacks]: #drawbacks

This tooling system is the default for javascript flavour projects. If we are working with an external partner where they have preferred tooling we should not enforce this approach.

A drawback to this approach is it asks developers for code in a potentially foreign style but after the initial mindset shift the benefits greatly overpower the drawbacks.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Prettier is fast becoming _the_ way to format code on JavaScript projects as mentioned before, it puts a hard stop to all on-going debates over styles. Prettier is also 100% automatic so there is zero mental effort to adopt.

There are alternatives like [EditorConfig](https://editorconfig.org/?ref=stackshare) but this only covers a small subset of what is capable with Prettier.

ESLint on the other hand helps developers spot basic potential errors in their code. Alternatives include JSLint but the ruleset is far too strict and is very difficult to get code to pass. Another alternative is JSHint but does not have great support for future facing code (ESNext).

# Prior art
[prior-art]: #prior-art

Using ESLint and Prettier is very common on projects, though our exact setup especially with the `@signal-noise/stylelint-scales` StyleLint rules is yet to be seen.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

One element that needs to be explored and proved rigorously in production is the use of some of our StyleLint rules. Including:

- Does the use of `stylelint-config-rational-order` cause a headache as when used in conjunction with StyleLint processors it removed the ability to automatically `--fix` errors.
- Is `@signal-noise/stylelint-scales` versatile enough. Is a compound rule needed to effectively restrain design systems?

# Future possibilities
[future-possibilities]: #future-possibilities

It would be good to expand upon this RFC over time when we find ourselves consistently using other plugins and extensions across multiple projects.
