# Testing JavaScript

These are my notes from the [Testing JavaScript][6] course by Kent C. Dodds.

## [Static analysis testing JavaScript applications][5]

### ESLint

[ESLint][0] is a static code analysis tool. This means it allows us to discover
problems with our JavaScript code without executing it.

#### Install ESLint

`npm install --save-dev eslint will` add eslint to your `package.json` as a dev
dependency.

#### Configure ESLint

We create an `.eslintrc` file to configure our ESLint.

```JavaScript
{

    "parserOptions": {
      "ecmaVersion": 2018 // specifies that we are writing the latest version of javascript
    },
    "extends": ["eslint:recommended"], // uses the built in eslint configurations
    "rules": {
      "no-console": "off" // means we don't get 'unexpected console statement' errors
    },
    "env": {
      "browser": true
    }
}
```

#### Add script to run ESLint

In the `package.json` we can add a script to run ESLint on our `src` folder..

```JavaScript
  "scripts": {
    "lint": "eslint src"
  },
```

Now to run ESLint we can do `npm run lint`.

#### Enable an ESLint plugin

The plugin allows us to see errors as we are coding, without having to
`npm run lint` in the terminal. I used the plugin named 'ESLint - Integrates
ESLint JavaScript into VS Code by Dirk Baeumer'.

### Prettier

[Prettier][1] is a code formatter that formats our code for us when we save
changes. It allows us to specify whether we want trailing commas, single or
double quotes, tab width, etc.

#### Install Prettier

Install prettier as a dev dependency with `npm install --save-dev prettier`.

#### Add script to run Prettier

In the `package.json` file we can include a new script to run Prettier.

```JavaScript
    "format": "prettier --write \"**/*.+(js|jsx|json|yml|yaml|css|less|scss|ts|tsx|md|graphql|mdx)\"" // prettier will format and save the changes to any file in the project with these extensions

```

#### Install Prettier plugin and enable format on save

The Prettier plugin allows us to perform Prettier format operations as we save
our code changes. To do this we add the plugin. I used 'Prettier - Code
formatter, VS Code plugin for prettier by Esben Petersen'.

We can then edit our `settings.json` to enable the format on save.

```JavaScript
"editor.formatOnSave": true
```

#### Configure Prettier

We add a configuration file to our project called `.prettierrc` to tell Prettier
how we want it to format our code.

We can use the [Prettier Playground][3] to generate our options, such as whether
we want trailing commas, tabs, etc.

We also add a `.prettierignore` file to make sure Prettier does not try and
format files we don't want it to, such as our node modules.

### When ESLint and other configurations conflict with Prettier

Sometimes, ESLint or other configurations will show errors or red underlines
beneath things that Prettier will fix for us on save.

To avoid the red underlines, we can extend our configuration to automatically
disable all the rules that Prettier renders irrelevant by installing
[eslint-config-prettier][2].

#### Install eslint-config-prettier

We can `npm install --save-dev eslint-config-prettier`.

Then in our `.eslintrc` we can add `eslint-config-prettier`.

```JavaScript
  "extends": ["eslint:recommended", "eslint-config-prettier"], // the rules at the end of the array take priority
```

#### Which rules take priority over others

The configurations that come at the end of the 'extends' arrray will win in a
conflict with the configurations that come first in the array, so in this case
`eslint-config-prettier` will win over `eslint:recommended`.

Furthermore, rules specified by myself in the `.eslintrc` will win in a conflict
with those in 'extends', such as the no-console rule below:

```JavaScript
  "rules": {
    "no-console": "off" // rules specified by myself take priority over eslint:recommended", "eslint-config-prettier
  },
```

### Flow

[Flow][4] is a tool for checking the types of variables in your application.

#### Installing Flow

We install Flow as a dev dependency with `npm install --save-dev flow-bin`.

#### Add script to run Flow

To our `package.json` scripts we simply add `"flow": "flow"`. We can then run
`npm run flow`.

### Unexpected token errors

ESLint doesn't understand Flow, so it will throw errors when using Flow syntax.

To fix this, we `npm install --save-dev babel-eslint`, and add
`parser: babel-eslint` to our `.eslintrc` file.

I also was receiving the following error in my Flow files: "types can only be
used in a .ts file". To fix that I added the Flow Language support plugin and
disabled TypeScript. Solution outlined in this [Stack Overflow][8].

### Handling projects with multiple people

What if someone working on our project doesn't have Prettier, ESLint or Flow set
up. How can we ensure that their code adheres to the rules for our project?

We can introduce a validation script to make sure things are linted, formatted
and flow-checked.

Here, our validation script runs our lint script, flow script and lists any
prettier descrepencies.

```JavaScript
  "scripts": {
    "lint": "eslint src",
    "format": "npm run prettier -- --write",
    "prettier": "prettier \"**/*.+(js|jsx|json|yml|yaml|css|less|scss|ts|tsx|md|graphql|mdx)\"", // prettier is reused in both the format and validate script
    "validate": "npm run lint && npm run prettier -- --list-different && npm run flow" // list-different lists any issues that our format script would have fixed
  },
```

But how do we ensure users run the validate script? That's where Husky comes in.

### Husky

The above validation script can be automatically run when people commit their
code by setting up a pre-commit hook with [Husky][7].

#### Installing Husky

We install Husky with `npm install --save-dev husky`. Note you need to have
initialized a Git repository before installing Husky, as it puts a pre-commit
file in our `.git` folder.

#### Adding the pre-commit script

We add `"precommit": "npm run validate"` to our `package.json` scripts, so that
our validate script runs when people attempt to commit their code.

The commit will fail if there are any lint, prettier or flow errors, and the
person will have to go and fix them.

But it turns out for ESLint and Prettier, we can fix the errors for them.

### Auto fixing ESLint and Prettier errors

We can automatically fix ESLint and Prettier errors when a person commmits their
code, rather than telling them where the errors are and having to fix it
themselves. Flow however still needs to be run across all files, regardless of
which ones have changed. To do this, we use `lint-staged`.

#### Installing lint-staged

We can run `npm install --save-dev lint-staged`.

#### Editing our pre-commit script

We can now edit our pre-commit script to run lint-staged and flow.

```JavaScript
"precommit": "lint-staged && npm run flow"
```

#### Configure lint-staged

We add a `.lintstagedrc` file to our project and tell it to run ESLint for all
files that end in .js. and Prettier for all the files that can be prettified.

```JavaScript
{
  "linters": {
    "*.js": [
      "eslint"
    ],
    "*.+(js|jsx|json|yml|yaml|css|less|scss|ts|tsx|md|graphql|mdx)": [
      "prettier --write",
      "git add"
    ]
  }
}
```

Now, when a user who doesn't have ESLint or Prettier configured tries to commit
code that can be automatically formatted for them, our pre-commit script will do
so.

## [JavaScript Mocking Fundamentals][9]

### jest.fn()

This is how `jest.fn` works under the hood:

```JavaScript
function fn(impl) {
  const mockFn = (...args) => {
    mockFn.mock.calls.push(args)
    return impl(...args)
  }
  mockFn.mock = {calls: []}
  return mockFn
}
```

`fn` accepts an implementation function, `impl`, and returns a mock function,
`mockFn`.

When the code that you're testing is executed, `mockFn` gets called in place of
your real function (see how `getWinner` is mocked in the usage example below).

When `mockFn` gets called it does two things with the arguments that it
receives:

1. Calls `impl` with the arguments
2. Keeps track of all the arguments it's called with in `mockFn.mock.calls` (see
   why this is useful in the expect example below)

#### fn usage example:

```JavaScript
test('returns winner', () => {
  // keep hold of the original implementation
  const originalGetWinner = utils.getWinner
  // mock getWinner
  utils.getWinner = jest.fn((p1, p2) => p1)

  // make the test code run
  const winner = thumbWar('Kent C. Dodds', 'Ken Wheeler')

  expect(winner).toBe('Kent C. Dodds')
  // make assertions about what the mock is called with
  expect(utils.getWinner.mock.calls).toEqual([
    ['Kent C. Dodds', 'Ken Wheeler'],
    ['Kent C. Dodds', 'Ken Wheeler']
  ])

  // reassign getWinnerr to its original value,
  // so the mock implementation does not leak into subsequent tests
  utils.getWinner = originalGetWinner
})
```

### jest.spyOn

With `jest.fn()` we had to keep track of the mock function's original
implementation, so that we could restore it later.

With `jest.spyOn`, keeping track of the original implementation is no longer
necessary, as this is done for us.

This is how `jest.spyOn` works under the hood (notice how internally it still
calls `fn`):

```JavaScript
function spyOn(obj, prop) {
  const originalValue = obj[prop]
  obj[prop] = fn()
  obj[prop].mockRestore = () => (obj[prop] = originalValue)
}

function fn(impl = () => {}) {
  const mockFn = (...args) => {
    mockFn.mock.calls.push(args)
    return impl(...args)
  }
  mockFn.mock = {calls: []}
  mockFn.mockImplementation = newImpl => (impl = newImpl)
  return mockFn
}
```

`spyOn` is a function that takes an object and a prop - the prop being the
function that you are mocking.

`spyOn` then does 3 things:

1.  Keep track of the original value of the prop
2.  Mock the prop by calling `fn`
3.  Expose a `mockRestore` function, which reassigns the prop to its original
    value

The implementation of `fn` is now slightly different to above. It receives a
default `impl` (an empty function), and adds a `mockImplementation` property
`mockFn` (the function being mocked).

#### spyOn usage example:

```JavaScript
const thumbWar = require('../thumb-war')
const utils = require('../utils')

test('returns winner', () => {
  // make utils.getWinner a mocked function
  jest.spyOn(utils, 'getWinner')
  // mock the implementation
  utils.getWinner.mockImplementation((p1, p2) => p1)

  const winner = thumbWar('Kent C. Dodds', 'Ken Wheeler')
  expect(winner).toBe('Kent C. Dodds')
  expect(utils.getWinner.mock.calls).toEqual([
    ['Kent C. Dodds', 'Ken Wheeler'],
    ['Kent C. Dodds', 'Ken Wheeler']
  ])

  // restore getWinner to its original value
  utils.getWinner.mockRestore()
})
```

## [Configure Jest for testing JavaScript Applications][10]

### Installing Jest

To install Jest into an existing project:

```
npm install --save-dev jest
```

In your `package.json` scripts add:

```
"test": "jest
```

Create a folder called `__tests__` (Jest will look here for tests to run) and a
new test file.

To run your tests you can do `npm run test`, `npm test`, or `npm t`.

To make ESLint play nicely with Jest, in your `.eslintrc` add:

```
  "env": {
    "jest": true
  }
```

[0]: https://eslint.org/docs/about
[1]: https://prettier.io/
[2]: https://www.npmjs.com/package/eslint-config-prettier
[3]: https://prettier.io/playground/
[4]: https://flow.org/en/
[5]:
  https://testingjavascript.com/courses/static-analysis-testing-javascript-applications
[6]: https://testingjavascript.com/
[7]: https://www.npmjs.com/package/husky
[8]:
  https://stackoverflow.com/questions/48859169/js-types-can-only-be-used-in-a-ts-file-visual-studio-code-using-ts-check
[9]: https://testingjavascript.com/courses/javascript-mocking-fundamentals
[10]:
  https://testingjavascript.com/courses/configure-jest-for-testing-javascript-applications
