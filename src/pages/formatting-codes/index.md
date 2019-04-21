---
title: Code Prettier
date: '2019-04-15'
spoiler: How to format codes.
---

There are many steps that every front-end developers and maintainers should know, they are:
 - [How to format codes](https://overafaik.com/formatting-codes)
 - [How to report on patterns found in ECMAScript/JavaScript code](https://overafaik.com/linting-codes)
 - [How to check object types](https://overafaik.com/flow-type-check)
 - [How to test everything](https://overafaik.com/jest)
 - [How to build the project](https://overafaik.com/build-project)
 - [How to use CI](https://overafaik.com/continous-integration)
 - How forced contributors to staying on contributing guidelines?

## No need to discuss style in code review
Sometimes you want to format your code with press save but there are many IDE and code formatters with many options such as `Wrap Line Length` and maybe all contributors don't want to use the same value of them.

so, What should we do?

There is an opinionated code formatter that support many languages and we can use it as a task in script command line.It is [Prettier](https://prettier.io/) with few options.

It can be used global. 
```
yarn global add prettier
```

but I recommend to use it locally and run it without any arguments. Although, Prettier ships with a handful of customizable format options, usable in both the CLI and API. I'd like to use general options because it helps me to keep the same format. To do this we need to create a file as `.prettierrc.js`.
```
module.exports = {
  bracketSpacing: false,
  singleQuote: true,
  jsxBracketSameLine: true,
  trailingComma: 'es5',
  printWidth: 80,
  parser: 'babylon',

  overrides: [
    {
      files: [
        // Internal forwarding modules
        'packages/*/*.js',
        // Source files
        'packages/*/src/**/*.js',
      ],
      options: {
        trailingComma: 'all',
      },
    },
  ],
};

```
You can see all options in https://prettier.io/docs/en/options.html

Now, maybe you think your project will have so many js files and this task takes too long. Don't worry!

In this situation there is a interesting way. Prettier is a `NPM` package and we can import and use it in our way. please see below example:

```javascript
//index.js
const execFileSync = require('child_process').execFileSync;
const chalk = require('chalk');
const glob = require('glob');
const prettier = require('prettier');
const fs = require('fs');
const prettierConfigPath = require.resolve('./.prettierrc');

const exec = (command, args) => {
  console.log('> ' + [command].concat(args).join(' '));
  const options = {
    cwd: process.cwd(),
    env: process.env,
    stdio: 'pipe',
    encoding: 'utf-8',
  };
  return execFileSync(command, args, options);
};

const execGitCmd = args =>
  exec('git', args)
    .trim()
    .toString()
    .split('\n');

const listChangedFiles = () => {
  const mergeBase = execGitCmd(['merge-base', 'HEAD', 'master']);
  return new Set([
    ...execGitCmd(['diff', '--name-only', '--diff-filter=ACMRTUB', mergeBase]),
    ...execGitCmd(['ls-files', '--others', '--exclude-standard']),
  ]);
};


const mode = process.argv[2] || 'check';
const shouldWrite = mode === 'write' || mode === 'write-changed';
const onlyChanged = mode === 'check-changed' || mode === 'write-changed';

const changedFiles = onlyChanged ? listChangedFiles() : null;
let didWarn = false;
let didError = false;

const files = glob
  .sync('**/*.js', {ignore: '**/node_modules/**'})
  .filter(f => !onlyChanged || changedFiles.has(f));

if (!files.length) {
  return;
}

files.forEach(file => {
  const options = prettier.resolveConfig.sync(file, {
    config: prettierConfigPath,
  });
  try {
    const input = fs.readFileSync(file, 'utf8');
    if (shouldWrite) {
      const output = prettier.format(input, options);
      if (output !== input) {
        fs.writeFileSync(file, output, 'utf8');
      }
    } else {
      if (!prettier.check(input, options)) {
        if (!didWarn) {
          console.log(
            '\n' +
              chalk.red(
                `  This project uses prettier to format all JavaScript code.\n`
              ) +
              chalk.dim(`    Please run `) +
              chalk.reset('yarn prettier-all') +
              chalk.dim(
                ` and add changes to files listed below to your commit:`
              ) +
              `\n\n`
          );
          didWarn = true;
        }
        console.log(file);
      }
    }
  } catch (error) {
    didError = true;
    console.log('\n\n' + error.message);
    console.log(file);
  }
});

if (didWarn || didError) {
  process.exit(1);
}
```

What happened? Let me explain:

In this example, as the beginning, I need to get changes. To do this I used `git` commands I mean *diff* with 'merge-base', 'HEAD', 'master' and *ls-files*. Then, checked out with `prettier.check` method and `.prettierrc.js` options.

Finlly, Requires two commands in `package.json` as bellow:

```json
  "scripts":{
    "prettier": "node ./scripts/prettier/index.js write-changed",
    "prettier-all": "node ./scripts/prettier/index.js write"
  }
```

