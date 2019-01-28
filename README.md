# Testing JavaScript

## Static code analysis

note on what is static code analysis

### ESLint

[ESLint][0] is a static code analysis tool. This means it allows us to discover
problems with our JavaScript code without executing it.

#### Install ESLint

`npm install --save-dev eslint will` add eslint to your `package.json` as a dev
dependency.

note on dev dependencies

#### Configure ESLint

we create a `.eslintrc` file to configure our ESLint.

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

The plugin allows us to see errors as we are coding, without having to run the
`npm run lint` in the terminal. I used 'ESLint - Integrates ESLint JavaScript
into VS Code by Dirk Baeumer'.

### Prettier

[Prettier][1] is a code formatter that formats our code for us when we save
changes.

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

```JavaScript
npm install --save-dev eslint-config-prettier.
```

Then in our `.eslintrc` we can add `eslint-config-prettier`.

```JavaScript
  "extends": ["eslint:recommended", "eslint-config-prettier"], // the rules at the end of the array take priority
```

#### Which rules take priority over others

The configurations that come at the end of 'extends' will win in a conflict with
the configurations that come before it, so in this case `eslint-config-prettier`
will win over `eslint:recommended`.

Furthermore, rules specified by myself in the `.eslintrc` will win in a conflict
with those in 'extends'.

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

### Husky

The above validation script can be automatically run when people commit their
code by setting up a pre-commit hook with Husky.

#### Installing Husky

We install Husky with `npm install --save-dev husky`. Note you need to have
initialized a Git repository before installing Husky, as it puts a pre-commit
file in our `.git` folder.

#### Adding the pre-commit script

We add `"precommit": "npm run validate"` to our `package.json` scripts, so that
our validate script runs when people attempt to commit their code.

The commit will fail if there are any lint, prettier or flow errors.

### Auto fixing ESLint and Prettier errors

We can automatically fix ESLint and Prettier errors when a person commmits their
code, rather than telling them where the errors are and having to fix it
themselves. Flow however still needs to be run across all files, regardless of
which ones have changed.

#### Installing lint-staged

We can run `npm install --save-dev lint-staged`.

#### Editing our pre-commit script

We can now edit our pre-commit script to run lint-staged and flow.

```JavaScript
"precommit": "lint-staged && npm run flow"
```

#### Configure lint-staged

We add `.lintstagedrc` to our project and tell it to run ESLint for all files
that end in .js. and Prettier for all the files that can be prettified.

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

[0]: https://eslint.org/docs/about
[1]: https://prettier.io/
[2]: https://www.npmjs.com/package/eslint-config-prettier
[3]: https://prettier.io/playground/
[4]: https://flow.org/en/
