# Cypress Image Snapshot

Cypress Image Snapshot binds [jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot)'s image diffing logic to [Cypress.io](https://cypress.io) commands.

[![build-and-test](https://github.com/emerson-eps/cypress-image-snapshot/actions/workflows/build-and-test.yml/badge.svg)](https://github.com/emerson-eps/cypress-image-snapshot/actions/workflows/build-and-test.yml)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Installation](#installation)
  - [TypeScript](#typescript)
- [Usage](#usage)
  - [In your tests](#in-your-tests)
    - [Options](#options)
  - [Updating snapshots](#updating-snapshots)
  - [Preventing failures](#preventing-failures)
  - [Requiring snapshots to be present](#requiring-snapshots-to-be-present)
- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Contributing](#contributing)
  - [Setup](#setup)
  - [Working on the plugin](#working-on-the-plugin)
    - [open](#open)
    - [run](#run)
      - [Note on environment variables](#note-on-environment-variables)
- [Forked from `jaredpalmer/cypress-image-snapshot`](#forked-from-jaredpalmercypress-image-snapshot)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation

Install with your chosen package manager

```bash
# yarn
yarn add --dev @emerson-eps/cypress-image-snapshot

# npm
npm install --save-dev @emerson-eps/cypress-image-snapshot
```

Next, import the plugin function and add it to the [`setupNodeEvents` function](https://docs.cypress.io/guides/references/configuration#setupNodeEvents):

```ts
// cypress.config.ts

import {defineConfig} from 'cypress'
import {addMatchImageSnapshotPlugin} from '@emerson-eps/cypress-image-snapshot/plugin'

export default defineConfig({
  e2e: {
    setupNodeEvents(on) {
      addMatchImageSnapshotPlugin(on)
    },
  },
})
```

Add the command to your relevant [support file](https://docs.cypress.io/guides/core-concepts/writing-and-organizing-tests#Support-file):

```ts
// cypress/support/e2e.ts

import {addMatchImageSnapshotCommand} from '@emerson-eps/cypress-image-snapshot'

addMatchImageSnapshotCommand()

// can also add any default options to be used
// by all instances of `matchImageSnapshot`
addMatchImageSnapshotCommand({
  failureThreshold: 0.2,
})
```

### TypeScript

TypeScript is supported so any reference to `@types/cypress-image-snapshot` can be removed from your project

Ensure that the types are included in your `tsconfig.json`

```
{
  "compilerOptions": {
    // ...
  },
  "include": ["@emerson-eps/cypress-image-snapshot"]
}
```

## Usage

### In your tests

```ts
describe('Login', () => {
  it('should be publicly accessible', () => {
    cy.visit('/login');

    // snapshot name will be the test title
    cy.matchImageSnapshot();

    // snapshot name will be the name passed in
    cy.matchImageSnapshot('login');

    // options object passed in
    cy.matchImageSnapshot({
      failureThreshold: 0.4
      blur: 10
      timeout: 3000
    });

    // match element snapshot
    cy.get('#login').matchImageSnapshot();
  });
});
```

#### Options

The options object combines jest-image-snapshot and Cypress screenshot configuration.

- [jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot#%EF%B8%8F-api)
- [Cypress screenshot](https://docs.cypress.io/api/commands/screenshot#Arguments)

```ts
cy.matchImageSnapshot({
  // options for jest-image-snapshot
  failureThreshold: 0.2,
  comparisonMethod: 'ssim',

  // options for Cypress.screenshot()
  capture: 'viewport',
  blackout: ['.some-element'],
})
```

New options have been implemented to make the function recursive and to retry the snapshots:

```ts
cy.matchImageSnapshot({
  // Time it takes for the snapshot command to time out if the snapshot is not correct
  timeout: 5000,

  //Sets a delay between recursive snapshots
  delayBetweenTries: 1000,
})
```

### Updating snapshots

Run Cypress with `--env updateSnapshots=true` in order to update the base image files for all of your tests.

### Requiring snapshots to be present

Run Cypress with `--env requireSnapshots=true` in order to fail if snapshots are missing. This is useful in continuous integration where snapshots should be present in advance.

## How it works

The workflow of `cy.matchImageSnapshot()` when running Cypress is:

1.  Take a screenshot with `cy.screenshot()` named according to the current test.
2.  Check if a saved snapshot exists in `<rootDir>/cypress/snapshots` and if so diff against that snapshot.
3.  If there is a resulting diff, save it to `<rootDir>/cypress/snapshots/__diff_output__`.

## Requirements

Tested on Cypress 10.x, 11.x and 12.x

Cypress must be installed as a peer dependency

## Contributing

### Setup

- Clone the repository and install the yarn dependencies with `yarn install`
- Ensure that Docker is setup. This is necessary for generating/updating snapshots
- Using [Volta](https://volta.sh/) is recommended for managing Node and Yarn versions. These are
  automatically picked up from the `package.json`
- Commits should be based on [conventional-changelog](https://github.com/pvdlg/conventional-changelog-metahub#commit-types)

### Working on the plugin

To make it easier to test whilst developing there are a few simple
Cypress tests that validate the plugin. There are two ways to run these tests:

#### open

`yarn test:open`

In open mode the tests run in Electron and ignore any snapshot failures. This is
due to the rendering differences on developer machines vs CI. Here there is also
verbose output sent to the test runner console to aid debugging.

**Note** here that the yarn script above will re-build the plugin each time. This is
necessary because the tests are run against the output in the `dist` directory
to ensure parity between the built package on NPM.

Ensure that the command is run each time changes need to be tested in Cypress

#### run

- `yarn docker:build`
- `yarn docker:run`

The commands here ensure that the tests are run inside a Docker container that
matches the CI machine. This allows images to be generated and matched correctly
when running the tests in Github Actions.

##### Note on environment variables

It is necessary to have two environment variables defined by default before
running the tests in Docker:

- `CYPRESS_updateSnapshots=false`
- `CYPRESS_debugSnapshots=false`

It's recommended that these are loaded into the shell with something like [direnv](https://direnv.net/)

Then they can be overridden as needed:

```
CYPRESS_updateSnapshots=true yarn docker:run
```

## Forked from `@simonsmith/cypress-image-snapshot`

This is a rewrite of the plugin from [Simon Smith](https://github.com/simonsmith/cypress-image-snapshot).
