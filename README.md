# `interpayshares`

The Interpayshares Module System is an open [ecosystem of modules](https://github.com/payshares/interpayshares/blob/master/docs/learn/module-list.md) that aims to make it easy to build a web application on the Payshares network. This repository (`interpayshares`) contains a command line tool that standardizes the build process for Payshares web applications based on the module configuration.

Read the [introductory blog post](https://www.payshares.org/blog/developer-preview-interpayshares-module-system/), [get started](https://github.com/payshares/interpayshares/blob/master/docs/learn/readme.md) or take a look at [interpayshares-client](https://github.com/payshares/interpayshares-client) to see the system in action.

# Overview

## Installation

```bash
npm install -g interpayshares
```

## Commands

Run these commands in your project home directory:
* `interpayshares build` - Builds a project
* `interpayshares develop` - Builds & watch project
* `interpayshares clean` - Remove temporary files and builds
* `interpayshares stroopy` - Prints [Stroopy](https://www.payshares.org/stories/adventures-in-galactic-consensus-chapter-1/)

## Guides

The [docs](docs) directory of this git repository contains a set of development guides to help you work within the IMS.

## Publishing to npm
```
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease]
npm publish
```
npm >=2.13.0 required.
Read more about [npm version](https://docs.npmjs.com/cli/version).
