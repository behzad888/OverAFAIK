---
title: Code Linting
date: '2019-04-16'
spoiler: How to reporting on patterns found in ECMAScript/JavaScript code.
---

There are many steps that every front-end developers and maintainers should know, they are:
 - [How to formatting codes](https://overafaik.com/formatting-codes)
 - [How to reporting on patterns found in ECMAScript/JavaScript code](https://overafaik.com/linting-codes)
 - [How to check object types](https://overafaik.com/flow-type-check)
 - [How to test everything](https://overafaik.com/jest)
 - [How to build the project](https://overafaik.com/build-project)
 - [How to use CI](https://overafaik.com/continous-integration)
 - How forced contributors to staying on contributing guidelines?

## How to reporting on patterns found in ECMAScript/JavaScript code
What does this means? maybe you heared about **lint** before. what really is this?

**Lint**, or a **linter**, is a tool that analyzes source code to flag programming errors, bugs, stylistic errors, and suspicious constructs.

There is a powerful library to analyze source code named **JSLint** for *.js* files and **TSLint** for *.ts* files.Also there is another linter for ECMAScript that goal is to provide a pluggable linting utility for JavaScript. Note that, these libraries used for *.jsx* and *.tsx* too. except for *node js* libraries, there are also few extensions for IDEs such as vscode to use but as I said before I want to stay on the same configuration to handle changes. so, I recommend to use it locally.

In this example I want to use ESLint and explain how it works AFAIK.

To do this make sure:
 - install ESLint 
 - To have configuration file `.eslintrc`
 
 ```
module.exports = {
  // Stop ESLint from looking for a configuration file in parent folders
  'root': true,

  plugins: [],

  parser: 'espree',

  parserOptions: {},

  rules: {},

  overrides:[]
  
  globals: {},
};
 ```

For more about advanced configuration please see https://eslint.org/docs/user-guide/getting-started

Now, what I've done?

I want to use `jest`, `react` plugins to linting so:

```
  plugins: [
    'jest',
    'no-for-of-loops',
    'react'
  ]
```

As you see, I used `espree` for prser option. The goal of `espree` is to produce output that is similar to [Esprima](http://esprima.org/) with a similar API so that it can be used in place of Esprima. Actually,Esprima is a high performance, standard-compliant ECMAScript parser written in ECMAScript.

Now, we can define parser options such as:

```
  parserOptions: {
    ecmaVersion: 2017,
    sourceType: 'script',
    ecmaFeatures: {
      experimentalObjectRestSpread: true,
    },
  }
```

before continue, I'd like to emphasize that the rules have a structure as below:

```
  "rules": {
        "semi": ["error", "always"],
        "quotes": ["error", "double"]
    }
```
The names "semi" and "quotes" are the names of rules in ESLint. The first value is the error level of the rule and can be one of these values:

 - "off" or 0 - turn the rule off
 - "warn" or 1 - turn the rule on as a warning (doesn’t affect exit code)
 - "error" or 2 - turn the rule on as an error (exit code will be 1)

I don't know but I'd like to say it :| .Although, you can see it on ESLint documentation.

Now, There are many rules that allow us control the source code.

```
'accessor-pairs': 0,
    'brace-style': [2, '1tbs'], //enforce consistent brace style for blocks
    'comma-dangle': [2, 'always-multiline'], //require or disallow trailing commas
    'consistent-return': 0, //require return statements to either always or never specify values
    'dot-location': [2, 'property'], //enforce consistent newlines before and after dots
    //...

    // We apply these settings to files that should run on Node.
    // They can't use JSX or ES6 modules, and must be in strict mode.
    // They can, however, use other ES6 features.
    // (Note these rules are overridden later for source files.)
    'no-var': 2,
    strict: 2,

    // React & JSX
    // Our transforms set this automatically
    'react/jsx-boolean-value': [2, 'always'],
    'react/jsx-no-undef': 2,
    // We don't care to do this
    'react/jsx-sort-prop-types': 0,
    'react/jsx-space-before-closing': 2,
    'react/jsx-uses-react': 2,
    'react/no-is-mounted': 0,
    // This isn't useful in our test code
    'react/react-in-jsx-scope': 2,
    'react/self-closing-comp': 2,
    // We don't care to do this
    'react/jsx-wrap-multilines': [2, {declaration: false, assignment: false}],

    // Prevent for...of loops because they require a Symbol polyfill.
    // You can disable this rule for code that isn't shipped (e.g. build scripts and tests).
    'no-for-of-loops/no-for-of-loops': 2,

```

If you're stricter than the default config like me. You should override a few rules:

```
  overrides: [
    {
      parser: 'espree',
      parserOptions: {
        ecmaVersion: 5,
        sourceType: 'script',
      },
      rules: {
        'no-var': 0,
        strict: 2,
      },
    },
    {
      // We apply these settings to the source files that get compiled.
      // They can use all features including JSX (but shouldn't use `var`).      
      parser: 'babel-eslint',
      parserOptions: {
        sourceType: 'module',
      },
      rules: {
        'no-var': 2,
        strict: 0,
      },
    },
    {
      files: ['**/__tests__/*.js'],
      rules: {
        // https://github.com/jest-community/eslint-plugin-jest
        'jest/no-focused-tests': 2,
      }
    }
  ],

```
Well, maybe you want to ignore some files for linting.No big deal! You can add everything that you want into `.eslintignore` file.

```
**/node_modules

# Build products
build/
```
I know there is great documentation for that but I'm an advocate developer with no blah blah blah :)
