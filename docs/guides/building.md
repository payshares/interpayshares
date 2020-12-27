---
id: build-process
title: Build Process
category: Getting Started
---

Build Process
===============

**Development of the `interpayshares` project has been paused. Please check out the [JS Payshares SDK](https://www.payshares.org/developers/js-payshares-sdk/learn/index.html) docs for how integrate with Payshares using JavaScript.**

This document explains what the build process looks like in Interpayshares.

You may want to begin with the [Getting Started](./) doc.

## Webpack

We're using [Webpack](http://webpack.github.io) for bundling and compilation because of its flexibility, performance and possible code optimizations. Webpack can not only bundle JS files but also stylesheets and this feature is very important in our modules environment.

## Javascript

Interpayshares is built using ECMAScript 6 standard of JavaScript. We're using [babel-loader](https://github.com/babel/babel-loader) to compile it to ES5 code. We strongly encourage you to use [ES6 Modules](https://github.com/lukehoban/es6features#modules) to load your files and export resources from your modules.

Interpayshares apps have three entry points:
* `vendor.es6` - loaded in `<head>` section of your app and contains common Interpayshares modules used by you app and modules.
* `head.es6` - loaded in `<head>` section of your app.
* `main.es6` -  loaded just before `</body>` of your app.

We will allow developers to create more entry points and change Webpack configuration in a near future.

## Stylesheets

_Read more about the css architecture in [solar css framework](https://github.com/payshares/solar) repo._

Interpayshares apps use the Solar css framework, a framework designed to enable modules to adapt to different themes with ease. Interpayshares apps and modules follow the architecture and style guide of the Solar framework.

Interpayshares Module System frontend's building blocks are widgets. Every widget styles are attached to its controller. This means that when developer decides to use a certain widget its styles gets loaded too! But what if an application developer wants to change how it looks like?

Interpayshares concatenates project files and then is using [node-sass](https://github.com/sass/node-sass) to compile it. We're concatenating stylesheets in this order:

1. Add in the theming layers of the [solar css framework](https://github.com/payshares/solar) and load the styles and libraries from each of the theme layers. These mixins and variables can be used later by modules and the app.
1. We check your application `styles` folder and concatenate all `*.header.scss` files. It's a great place to add your core styles.
1. We concatenate all styles of loaded widgets.
1. We check your application `styles` folder again but this time we concatenate all `*.footer.scss` files. It's a great place to overwrite widgets' styles.

## CLI

[`interpayshares`](https://github.com/payshares/interpayshares) is our CLI tool that performs all of the compilation steps. Use:

* `interpayshares develop` to open your application in a browser a see changes as you work.
* `interpayshares build` to build you application.

## Where to go from here?

<!-- this section is copied in other .md files in this docs folder -->
* Join our [Slack channel](http://slack.payshares.org/) to discuss Interpayshares and share ideas!
* Help us develop our [interpayshares-client](https://github.com/payshares/interpayshares-client).
* [Fix issues](https://github.com/issues?utf8=%E2%9C%93&q=is%3Aopen+repo%3Apayshares%2Finterpayshares+repo%3Apayshares%2Finterpayshares-core+repo%3Apayshares%2Finterpayshares-network+repo%3Apayshares%2Finterpayshares-network-widgets+repo%3Apayshares%2Finterpayshares-wallet+repo%3Apayshares%2Finterpayshares-sessions+repo%3Apayshares%2Finterpayshares-client) in Interpayshares modules.
* Develop your own module!
