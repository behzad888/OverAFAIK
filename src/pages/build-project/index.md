---
title: Building Project
date: '2019-04-21'
spoiler: How to build the project.
---

There are many steps that every front-end developers and maintainers should know, they are:
 - [How to formatting codes](https://overafaik.com/formatting-codes)
 - [How to reporting on patterns found in ECMAScript/JavaScript code](https://overafaik.com/linting-codes)
 - [How to check object types](https://overafaik.com/flow-type-check)
 - [How to test everything](https://overafaik.com/jest)
 - [How to build the project](https://overafaik.com/build-project)
 - [How to use CI](https://overafaik.com/continous-integration)
 - How forced contributors to staying on contributing guidelines?

Modern JavaScript Frameworks like Angular, React and Vue.js makes it very easy to build complex single page web applications. However, using a those frameworks is not mandatory and you can also go with plain and pure JavaScript. In this tutorial you you through building a web application with small pieces.

## What is Rollup

> [Rollup](https://rollupjs.org/) is a module bundler for JavaScript which compiles small pieces of code into something larger and more complex, such as a library or application. It uses the standardized ES module format for code, instead of previous idiosyncratic solutions such as CommonJS and AMD. ES modules let you freely and seamlessly combine the most useful individual functions from your favorite libraries. Rollup can optimize ES modules for faster native loading in modern browsers, or output a legacy module format allowing ES module workflows today.

There are many plugin that help you to build your project/library however you want.

```js
const {rollup} = require('rollup');
const babel = require('rollup-plugin-babel');
const commonjs = require('rollup-plugin-commonjs');
const prettier = require('rollup-plugin-prettier');
const replace = require('rollup-plugin-replace');
const stripBanner = require('rollup-plugin-strip-banner');
const resolve = require('rollup-plugin-node-resolve');
```

The First step that we need is to define a config for each plugin. Lets get started with `babel` config and compile codes/plugins to ES5:

```js
function getBabelConfig(filename) {
  let options = {
    exclude: '/**/node_modules/**',
    presets: [],
    plugins: [],
  };

  return Object.assign({}, options, {
    plugins: options.plugins.concat([
      // Concat your custom options here such as
      require('./wrap-warning-with-env-check'),
    ]),
  });  
}

```
For custom option you can see below. It helps us to wrap `warning()` calls in a `__DEV__` check so they are stripped from production. 

```js
//wrap-warning-with-env-check.js
// custom option for babel cofig
module.exports = function(babel, options) {
  const t = babel.types;

  const DEV_EXPRESSION = t.identifier('__DEV__');

  const SEEN_SYMBOL = Symbol('expression.seen');

  return {
    visitor: {
      CallExpression: {
        exit: function(path) {
          const node = path.node;

          // Ignore if it's already been processed
          if (node[SEEN_SYMBOL]) {
            return;
          }

          if (
            path.get('callee').isIdentifier({name: 'warning'}) ||
            path.get('callee').isIdentifier({name: 'warningWithoutStack'})
          ) {
            const condition = node.arguments[0];
            const newNode = t.callExpression(
              node.callee,
              [t.booleanLiteral(false)].concat(node.arguments.slice(1))
            );
            newNode[SEEN_SYMBOL] = true;
            path.replaceWith(
              t.ifStatement(
                DEV_EXPRESSION,
                t.blockStatement([
                  t.ifStatement(
                    t.unaryExpression('!', condition),
                    t.expressionStatement(newNode)
                  ),
                ])
              )
            );
          }
        },
      },
    },
  };
};
```

Next, We should get plugins to build and making bundles. I'd like to handle this as below:

```js
function wrapBundle(source, bundleType, globalName, filename, moduleType) {  
  
  //you can define many bundle types here
  
  return [bundleType](source, globalName, filename, moduleType) {
    return `/**
${license}
 *
 * @noflow
 * @preventMunge
 * @preserve-invariant-messages
 */

${source}`;
  },
}


function getPlugins(
  entry,
  externals,
  updateBabelOptions,
  filename,
  packageName,
  bundleType,
  globalName,
  moduleType,
  pureExternalModules
) {
  const isProduction = true; //We can hanlde dev mode even
  return [
    // Use Node resolution mechanism.
    resolve({
      skip: externals,
    }),
    // Remove license headers from individual modules
    stripBanner({
      exclude: 'node_modules/**/*',
    }),
    // Compile to ES5.
    babel(getBabelConfig(updateBabelOptions, bundleType)),
    // Remove 'use strict' from individual source files.
    {
      transform(source) {
        return source.replace(/['"]use strict['"']/g, '');
      },
    },
    // Turn __DEV__ and process.env checks into constants.
    replace({
      __DEV__: isProduction ? 'false' : 'true',
      __UMD__: isUMDBundle ? 'true' : 'false',
      'process.env.NODE_ENV': isProduction ? "'production'" : "'development'",
    }),
    // We still need CommonJS for external deps like object-assign.
    commonjs(),
    // Add the whitespace back if necessary.
    prettier({parser: 'babylon'}),
    // License and haste headers, top-level `if` blocks.
    {
      transformBundle(source) {
        return Wrappers.wrapBundle(
          source,
          bundleType,
          globalName,
          filename,
          moduleType
        );
      },
    },
  ].filter(Boolean);
}
```

Note that, There is some tip in above `getPlugin` function that could be useful. Rollup "runs" each statement to see what needs to be included. so, Rollup will exclude unused codes in output but sometimes we don't need to remove unused pure-module imports. There is a hack to handle this by adding:

```js
function stripUnusedImports(pureExternalModules) {
  return {
    name: 'strip-unused-imports',
    transformBundle(code) {
      pureExternalModules.forEach(module => {
        // Ideally this would use a negative lookbehind: (?<!= *)
        // But this isn't supported by the Node <= 8.9.
        // So instead we try to handle the most common cases:
        // 1. foo,bar=require("bar"),baz
        // 2. foo;bar = require('bar');baz
        // 3.   require('bar');
        const regExp = new RegExp(
          `([,;]| {2})require\\(["']${module}["']\\)[,;]`,
          'g'
        );
        code = code.replace(regExp, '$1');
      });
      return {code};
    },
  };
}

function getPlugin(...){
  //...
  stripUnusedImports(pureExternalModules),
  //...
}
```
Note that this plugin must be called after closure applies DCE if you used closure.

Another thing that could be useful is recording plugin bundle size: 

```js
const chalk = require('chalk');
const gzip = require('gzip-size');
function sizes(options) {
  return {
    name: 'sizes-plugin',
    ongenerate(bundle, obj) {
      const size = Buffer.byteLength(obj.code);
      const gzipSize = gzip.sync(obj.code);

      options.getSize(size, gzipSize);
    },
  };
};

function getPlugin(...){
  //...
  sizes({
    getSize: (size, gzip) => {
      console.log('GZip size = ' + gzip);
      console.log('size = ' + size);
    },
  }),
  //...
}
```

Finally, the last step is to determine rollup options:

```js
buildEverything(){
  await createBundle(bundle, 'BundleType');
}


function getPackageName(name) {
  if (name.indexOf('/') !== -1) {
    return name.split('/')[0];
  }
  return name;
}

function getFilename(name, globalName, bundleType) {  
  name = name.replace('/', '-');
  return `${name}.production.min.js`;  
}

function getPeerGlobals(externals, bundleType) {
  const peerGlobals = {};
  externals.forEach(name => {
    if (
      !knownGlobals[name] &&
      bundleType === 'BundleType'
    ) {
      throw new Error('Cannot build UMD without a global name for: ' + name);
    }
    peerGlobals[name] = knownGlobals[name];
  });
  return peerGlobals;
}

// Determines node_modules packages that are safe to assume will exist.
function getDependencies(bundleType, entry) {
  // Replaces any part of the entry that follow the package name (like  
  const packageJson = require(entry.replace(/(\/.*)?$/, '/package.json'));
  // Both deps and peerDeps are assumed as accessible.
  return Array.from(
    new Set([
      ...Object.keys(packageJson.dependencies || {}),
      ...Object.keys(packageJson.peerDependencies || {}),
    ])
  );
}

function getRollupOutputOptions(
  outputPath,
  format,
  globals,
  globalName,
  bundleType
) {
  const isProduction = isProductionBundleType(bundleType);

  return Object.assign(
    {},
    {
      file: outputPath,
      format,
      globals,
      freeze: !isProduction,
      interop: false,
      name: globalName,
      sourcemap: false,
    }
  );
}

async function createBundle(bundle, bundleType) {
  const filename = getFilename(bundle.entry, bundle.global, bundleType);
  const logKey =
    chalk.white.bold(filename) + chalk.dim(` (${bundleType.toLowerCase()})`);
  const format = `umd`;
  const packageName = getPackageName(bundle.entry);
  let resolvedEntry = require.resolve(bundle.entry);
      
  const peerGlobals = getPeerGlobals(bundle.externals, bundleType);
  const deps = getDependencies(bundleType, bundle.entry);
  let externals = Object.keys(peerGlobals);
  externals = externals.concat(deps);
  //determine pureExternalModules 
  pureExternalModules =  [];
 
 const rollupConfig = {
    input: resolvedEntry,
    treeshake: {
      pureExternalModules,
    },
    external(id) {
      const containsThisModule = pkg => id === pkg || id.startsWith(pkg + '/');
      const isProvidedByDependency = externals.some(containsThisModule);
      if (isProvidedByDependency) {
        return true;
      }
      return !!peerGlobals[id];
    },    
    plugins: getPlugins(
      bundle.entry,
      externals,
      bundle.babel,
      filename,
      packageName,
      bundleType,
      bundle.global,
      bundle.moduleType,
      pureExternalModules
    ),    
    legacy: true      
  };
  const [mainOutputPath, ...otherOutputPaths] =  [`build/${packageName}/${filename}`];
  const rollupOutputOptions = getRollupOutputOptions(
    mainOutputPath,
    format,
    peerGlobals,
    bundle.global,
    bundleType
  );

  console.log(`${chalk.bgYellow.black(' BUILDING ')} ${logKey}`);
  try {
    const result = await rollup(rollupConfig);
    await result.write(rollupOutputOptions);
  } catch (error) {
    console.log(`${chalk.bgRed.black(' OH NOES! ')} ${logKey}\n`);
    handleRollupError(error);
    throw error;
  }
  for (let i = 0; i < otherOutputPaths.length; i++) {
    await asyncCopyTo(mainOutputPath, otherOutputPaths[i]);
  }
  console.log(`${chalk.bgGreen.black(' COMPLETE ')} ${logKey}\n`);
}
```

I know its too long but so many useful, You can see it as a cheat sheet too. Now, add a command in your `package.json` to build your plugins. `"build": "node ./build.js"`