# Project Configuration: Dev, Test, Build

## Babel

Out of the box Redwood configures [Babel](https://babeljs.io/) so that you can write modern JavaScript and TypeScript without needing to worry about transpilation at all.
GraphQL tags, JSX, SVG imports—all of it's handled for you.

For those well-versed in Babel config, you can find Redwood's in [@redwoodjs/internal](https://github.com/redwoodjs/redwood/tree/main/packages/internal/src/build/babel).

### Configuring Babel

For most projects, you won't need to configure Babel at all, but if you need to you can configure each side (web, api) individually using side-specific `babel.config.js` files.

> **Heads up**
>
> `.babelrc{.js}` files are ignored.
> You have to put your custom config in the appropriate side's `babel.config.js`: `web/babel.config.js` for web and `api/babel.config.js` for api.

Let's go over an example.

#### Example: Adding Emotion

Let's say we want to add the styling library [emotion](https://emotion.sh), which requires adding a Babel plugin.

1. Create a `babel.config.js` file in `web`:
```shell
touch web/babel.config.js
```
<br />

2. Add the `@emotion/babel-plugin` as a dependency:
```shell
yarn workspace web add --dev @emotion/babel-plugin
```
<br />

3. Add the plugin to `web/babel.config.js`:
```js
// web/babel.config.js

module.exports = {
  plugins: ["@emotion"] // 👈 add the emotion plugin
}

// ℹ️ Notice how we don't need the `extends` property
```

That's it!
Now your custom web-side Babel config will be merged with Redwood's.

## Jest

Redwood uses [Jest](https://jestjs.io/) for testing.
Let's take a peek at how it's all configured.

At the root of your project is `jest.config.js`. 
It should look like this:

```js
// jest.config.js

module.exports = {
  rootDir: '.',
  projects: ['<rootDir>/{*,!(node_modules)/**/}/jest.config.js'],
}
```

This just tells Jest that the actual config files sit in each side, allowing Jest to pick up the individual settings for each.
`rootDir` also makes sure that if you're running Jest with the `--collectCoverage` flag, it'll produce the report in the root directory.

#### Web Jest Config

The web side's configuration sits in `./web/jest.config.js`

```js
const config = {
  rootDir: '../',
  preset: '@redwoodjs/testing/config/jest/web',
  // ☝️ load the built-in Redwood Jest configuartion
}

module.exports = config
```

> You can always see Redwood's latest configuration templates in the [create-redwood-app package](https://github.com/redwoodjs/redwood/blob/main/packages/create-redwood-app/template/web/jest.config.js).

The preset includes all the setup required to test everything that's going on in web: rendering React components and transforming JSX, automatically mocking Cells, transpiling with Babel, mocking the Router and the GraphQL client—the list goes on!
You can find all the details in the [source](https://github.com/redwoodjs/redwood/blob/main/packages/testing/config/jest/web/jest-preset.js).

#### Api Side Config

The api side is configured similarly, with the configuration sitting in `./api/jest.config.js`.
But the api preset is slightly different in that:

- it's configured to run tests serially (because Scenarios seed your test database)
- it has setup code to make sure your database is 1) seeded before running tests 2) reset between Scenarios

You can find all the details in the [source](https://github.com/redwoodjs/redwood/blob/main/packages/testing/config/jest/api/jest-preset.js).

## GraphQL Codegen

Redwood uses [GraphQL Code Generator](https://www.graphql-code-generator.com) to generate types for your GraphQL queries and mutations.

While the defaults are configured so that things JustWork™️, you can customize them by adding a `./codegen.yml` file to the root of your project.
Your custom settings will be merged with the built-in ones.

> If you're curious about the built-in settings, they can be found [here](https://github.com/redwoodjs/redwood/blob/main/packages/internal/src/generate/typeDefinitions.ts) in the Redwood source. Look for the `generateTypeDefGraphQLWeb` and `generateTypeDefGraphQLApi` functions.

For example, adding this `codegen.yml` to the root of your project will transform all the generated types to UPPERCASE:

```yml
# ./codegen.yml

config:
  namingConvention:
    typeNames: change-case-all#upperCase
```

For completeness, [here's the docs](https://www.graphql-code-generator.com/docs/config-reference/config-field) on configuring GraphQL Code Generator.
