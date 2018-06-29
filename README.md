# vue-cli-plugin-apollo

[![npm](https://img.shields.io/npm/v/vue-cli-plugin-apollo.svg) ![npm](https://img.shields.io/npm/dm/vue-cli-plugin-apollo.svg)](https://www.npmjs.com/package/vue-cli-plugin-apollo)
[![vue-cli3](https://img.shields.io/badge/vue--cli-3.x-brightgreen.svg)](https://github.com/vuejs/vue-cli)
[![apollo-2](https://img.shields.io/badge/apollo-2.x-blue.svg)](https://www.apollographql.com/)

**:rocket: Start building a Vue app with Apollo and GraphQL in 2 minutes!**

This is a vue-cli 3.x plugin to add Apollo and GraphQL in your Vue project.

<p>
  <a href="https://www.patreon.com/akryum" target="_blank">
    <img src="https://c5.patreon.com/external/logo/become_a_patron_button.png" alt="Become a Patreon">
  </a>
</p>

<br>

**:star: Features:**

- Automatically integrate [vue-apollo](https://github.com/Akryum/vue-apollo) into your Vue app
- Embed Apollo client config (upgradable)
  - Websockets
  - Client state with [apollo-link-state](https://github.com/apollographql/apollo-link-state)
- Included optional Graphql API Server (upgradable):
  - Dead simple GraphQL API sources generated into your project
  - Upgradable service running [apollo-server](https://www.apollographql.com/docs/apollo-server/)
  - Websocket subscriptions support
  - Optional automatic mocking
  - Optional [Apollo Engine](https://www.apollographql.com/engine) support
- Included optional example component with:
  - Watched query
  - Mutation
  - Realtime subscription using Websockets
- GraphQL validation using ESLint

## Table of contents

- [Getting started](#getting-started)
  - [GraphQL API Server](#graphql-api-server)
- [Injected Commands](#injected-commands)
- [Configuration](#configuration)
  - [Client state](#client-state)
  - [Authorization Header](#authorization-header)
  - [Plugin options](#plugin-options)
  - [Mocks](#mocks)
  - [Directives](#directives)
  - [Apollo Engine](#apollo-engine)
  - [Express middlewares](#express-middlewares)
- [Env variables](#env-variables)
- [Injected webpack-chain Rules](#injected-webpack-chain-rules)
- [Running the GraphQL server in production](#running-the-graphql-server-in-production)
- [Manual code changes](#manual-code-changes)

---

## Getting started

:warning: Make sure you have vue-cli 3.x.x:

```
vue --version
```

If you don't have a project created with vue-cli 3.x yet:

```
vue create my-new-app
```

Navigate to the newly created project folder and add the cli plugin:

```
cd my-new-app
vue add apollo
```

*:information_source: An example `ApolloExample.vue` component alongside some GraphQL query files will be added into your sources if you chose to include the examples.*

Start your app:

```
npm run serve
```

[Recommended VS Code extension](https://github.com/prismagraphql/vscode-graphql)

**Updating `vue-cli-plugin-apollo` will also update both Apollo Client and its configuration for you! :+1:**

Read the [vue-apollo doc](https://github.com/Akryum/vue-apollo).

### GraphQL API Server

If you enabled the GraphQL API Server, open a new terminal and start it:

```
npm run apollo
```

You can edit the files generated in the `./apollo-server` folder:

- `schema.graphql` contains the Schema written with the [schema definition language](https://github.com/facebook/graphql/blob/master/spec/Section%203%20--%20Type%20System.md).
- `resolvers.js` declares the [Apollo resolvers](https://www.apollographql.com/docs/graphql-tools/resolvers.html).
- `context.js` allows injecting a context object into all the resolvers (third argument).
- `mocks.js` defines the custom resolvers used for mocking ([more info](https://www.apollographql.com/docs/graphql-tools/mocking.html#Customizing-mocks)).
- `directives.js` defines the custom schema directives ([more info](https://www.apollographql.com/docs/graphql-tools/schema-directives.html))).

The server will be automatically restarted when a change is detected.

To run the server only once, use this command:

```
npm run run-graphql-api
```

**Updating `vue-cli-plugin-apollo` will also update the GraphQL Server service :+1:**

## Injected Commands

- **`vue-cli-service apollo:watch`**

  Run the GraphQL API server with info from `./apollo-server` and watch the files to restart itself automatically.

- **`vue-cli-service apollo:run`**

  Run the GraphQL API server with info from `./apollo-server` once.

## Configuration

### Plugin options

The GraphQL API Server can be configured via the `pluginOptions` in `vue.config.js`:

``` js
module.exports = {
  // Other options...
  pluginOptions: {
    // Apollo-related options
    apollo: {
      // Enable automatic mocking
      enableMocks: true,
      // Enable Apollo Engine
      enableEngine: true,

      /* Other options (with default values) */

      // Cross-Origin options
      cors: '*',
      // Requests timeout (ms)
      timeout: 120000,
      // Integrated apollo engine
      integratedEngine: true,
      // Base folder for the server source files
      serverFolder: './apollo-server',
      // Apollo server options (will be merged with the included default options)
      serverOptions: {
        // ...
      },
    },
  },
}
```

See [Apollo Server options](https://www.apollographql.com/docs/apollo-server/v2/api/apollo-server.html#constructor-options-lt-ApolloServer-gt).

### Client state

You can use [apollo-link-state](https://github.com/apollographql/apollo-link-state) for client-only local data with the `clientState` option of `createApolloClient`:

```js
import { createApolloClient } from 'vue-cli-plugin-apollo/graphql-client'

const options = {
  // ...

  clientState: {
    defaults: {
      connected: false,
    },
    resolvers: {
      Mutation: {
        connectedSet: (root, { value }, { cache }) => {
          const data = {
            connected: value,
          }
          cache.writeData({ data })
        },
      },
    },
  },
}

const { apolloClient } = createApolloClient(options)
```

Then you need to use the `@client` directive:

```graphql
query isConnected {
  connected @client
}
```

```graphql
mutation setConnected ($value: Boolean!) {
  connectedSet (value: $value) @client
}
```

### Authorization Header

By default, `createApolloClient` will retrieve the `Authorization` header value from `localStorage`. You can override this behavior with the `getAuth` option:

```js
const options = {
  // ...

  getAuth: (tokenName) => getUserToken(),
}

const { apolloClient } = createApolloClient(options)
```

If you use cookies, you can return `undefined`.

### Mocks

You can enable automatic mocking on the GraphQL API Server. It can be [customized](https://www.apollographql.com/docs/graphql-tools/mocking.html#Customizing-mocks) in the `./apollo-server/mocks.js` file generated in your project.

### Directives

You can add custom GraphQL directives in the `./apollo-server/directives.js` file ([documentation](https://www.apollographql.com/docs/graphql-tools/schema-directives.html)).

```js
const { SchemaDirectiveVisitor } = require('graphql-tools')

class PrivateDirective extends SchemaDirectiveVisitor {
  // ...
}

module.exports = {
  // Now you can use '@private' in the schema
  private: PrivateDirective
}
```

### Apollo Engine

[Apollo Engine](https://www.apollographql.com/engine) is a commercial product from Apollo. It enables lots of additional features like monitoring, error reporting, caching and query persisting.

Create a key at https://engine.apollographql.com (it's free!).

### Express middlewares

If you need to add express middlewares into the GraphQL server, you can create a `./apollo-server/server.js` file:

```js
const path = require('path')
const express = require('express')
const distPath = path.resolve(__dirname, '../../dist')

module.exports = app => {
  app.use(express.static(distPath))
}
```

## Env variables

- **`VUE_APP_GRAPHQL_HTTP`**

  The url to the graphql HTTP endpoint, default: `http://localhost:4000`

- **`VUE_APP_GRAPHQL_WS`**

  The url to the graphql Websockets endpoint for subscriptions, default: `ws://localhost:4000`

### With the GraphQL server enabled

- **`VUE_APP_GRAPHQL_PORT`**

  Port of the GraphQL API Server, default: `4000`

- **`VUE_APP_APOLLO_ENGINE_KEY`**

  API key for [Apollo Engine](https://engine.apollographql.com)

## Injected webpack-chain Rules

- `config.rule('gql')`

## Running the GraphQL server in production

```
cross-env NODE_ENV=production yarn run apollo:run --mode production
```

If your project is meant to be used as a package installed from npm, you will need to move `vue-cli-plugin-apollo` from the `devDependencies` field to `dependencies` in your `package.json` file.

## Manual code changes

In case the plugin isn't able to modify the file containing the root Vue instance:

Import the provider:

```js
import { createProvider } from './vue-apollo'
```

Then in the root instance, set the `provide` option:

```js
new Vue({
  el: '#app',
  // Add this line
  provide: createProvider().provide(),
})
```
