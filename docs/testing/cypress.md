# Why Cypress
Cypress is a great E2E testing tool. Here are a few great reasons to consider it:

* Isolated installation possible.
* Provides a nice interactive google chrome debug experience. This is very similar to how UI devs mostly work manually.
* Provides the ability to mock out backend XHR with fixtures, this allows easier data setup and tested error handling in real live application code.
* First class TypeScript support. Ships with TypeScript definitions out of the box.

## Installation

Create an e2e directory and install cypress and its dependencies for TypeScript transpiling:

```sh
mkdir e2e
cd e2e
npm init -y
npm install cypress webpack @cypress/webpack-preprocessor typescript ts-loader
```

> Here are a few reasons for creating a seperate `e2e` folder especially for cypress: 
* Creating a seperate directory or `e2e` makes it easier to isolate its `package.json` dependencies from the rest of your project. This results in less dependency conflicts.
* Testing frameworks have a habit of polluting the global namespace with stuff like `describe` `it` `expect`. It is best to keep the e2e `tsconfig.json` and `node_modules` in this special `e2e` folder to prevent global type definition conflicts.

Setup TypeScript `tsconfig.json` e.g. 

```json
{
  "compilerOptions": {
    "strict": true,
    "sourceMap": true,
    "module": "commonjs",
    "target": "es5",
    "lib": [
      "dom",
      "es6"
    ],
    "jsx": "react",
    "experimentalDecorators": true
  },
  "compileOnSave": false
}
```

Do a first dry run of cypress to prime the cypress folder structure. The Cypress IDE will open. You can close it after you see the welcome message.

```sh
npx cypress open
```

Setup cypress for transpiling typescript by editing `e2e/cypress/plugins/index.js` to match the following:

```js
const wp = require('@cypress/webpack-preprocessor')
module.exports = (on) => {
  const options = {
    webpackOptions: {
      resolve: {
        extensions: [".ts", ".tsx", ".js"]
      },
      module: {
        rules: [
          {
            test: /\.tsx?$/,
            loader: "ts-loader",
            options: { transpileOnly: true }
          }
        ]
      }
    },
  }
  on('file:preprocessor', wp(options))
}
```


Optionally add a few scripts to the `e2e/package.json` file:

```json
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run"
  },
```

## More description of key Files
Under the `e2e` folder you now have these files: 

* `/cypress.json`: Configure cypress. The default is empty and that is all you need.
* `/cypress` Subfolders: 
    * `/fixtures`: Test fixtures
        * Comes with `example.json`. Feel free to delete it. 
        * You can create simple `.json` files that can be used to provide sample data (aka fixtures) for usage across tests. 
    * `/integration`: All your tests. 
        * Comes with an `examples` folder. You can safely delete it.
        * Name tests with `.spec.ts` e.g. `something.spec.ts`. 
        * Feel free to create tests under subfolders for better organization e.g. `/someFeatureFolder/something.spec.ts`.

## First test 
* create a file `/cypress/integration/first.spec.ts` with the following contents: 

```ts
/// <reference types="cypress"/>

describe('google search', () => {
  it('should work', () => {
    cy.visit('http://www.google.com');
    cy.get('#lst-ib').type('Hello world{enter}')
  });
});
```

## Running in development
Open the cypress IDE using the following command.

```sh
npm run cypress:open
```

And select a test to run.

## Running on a build server

You can run cypress tests in ci mode using the following command.

```sh
npm run cypress:run
```

## Tip: Sharing code between UI and test
Cypress tests are compiled / packed and run in the browser. So feel free to import any project code into your test.

For example you can share Id values between UI and Tests to make sure the CSS selectors don't break:

```
import { Ids } from '../../../src/app/constants'; 

// Later 
cy.get(`#${Ids.username}`)
  .type('john')
```

## Tip: Waiting for an HTTP request
A lot of tests have been traditially brittle due to all arbitrary timeouts needed for XHRs that an application makes. `cy.server` makes it easy to 
* create an alias for backend calls
* wait for them to occur

e.g. 

```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load')
  .as('load') // create an alias

// Start test
cy.visit('/')

// wait for the call
cy.wait('@load') 

// Now the data is loaded
```

## Tip: Mocking an HTTP request response
You can also easily mock out a request response using `route`: 
```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load', /* Example payload response */{success:true})
```

## Tip: Mocking time 
You can use `wait` to pause test for some time e.g. to test automatic "you are about to be logged out" notification screen:.

```ts
cy.visit('/');
cy.wait(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

However it is recommended to mock time using `cy.clock` and forwarding time using `cy.tick` e.g. 

```ts
cy.clock();

cy.visit('/');
cy.tick(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

## Resources 
* Website: https://www.cypress.io/
* Write your first cypress test (gives a nice tour of the cypress IDE) : https://docs.cypress.io/guides/getting-started/writing-your-first-test.html
* Setting up a CI enviroment (e.g. the provided docker image that works out of the box with `cypress run`): https://docs.cypress.io/guides/guides/continuous-integration.html
* Recipes (Lists recipies with descriptions. Click on headings to navigate to the source code for the recipe): https://docs.cypress.io/examples/examples/recipes.html
