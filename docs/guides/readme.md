---
title: Overview
---

# `interpayshares`

**Development of the `interpayshares` project has been paused. Please check out the [JS Payshares SDK](https://www.payshares.org/developers/js-payshares-sdk/learn/index.html) docs for how integrate with Payshares using JavaScript.**

The Interpayshares Module System is an open [ecosystem of modules](https://github.com/payshares/interpayshares/blob/master/docs/module-list.md) that aims to make it easy to build a web application on the Payshares network. This repository (`interpayshares`) contains a command line tool that standardizes the build process for Payshares web applications based on the module configuration.

Read the [introductory blog post](https://www.payshares.org/blog/developer-preview-interpayshares-module-system/), [get started](https://github.com/payshares/interpayshares/blob/master/docs/readme.md) or take a look at [interpayshares-client](https://github.com/payshares/interpayshares-client) to see the system in action.

Getting started
===============

This document will guide you through main concepts and advantages of Interpayshares Module System and will help you develop your first Interpayshares application.

## Contents

1. [Why Interpayshares?](#why-interpayshares)
2. [Quick Overview](#quick-overview)
1. [Prerequisites](#prerequisites)
1. [Installing `interpayshares` command line tool](#installing-interpayshares-command-line-tool)
1. [Generating sample app](#generating-sample-app)
1. [Interpayshares application architecture](#interpayshares-application-architecture)
1. [Where to go from here?](#where-to-go-from-here)
2. [Get a Payshares testnet account](#get-a-payshares-testnet-account)
2. [List of modules](https://github.com/payshares/interpayshares/blob/master/docs/module-list.md)

## Why Interpayshares?

People often think of modularity in silos:
- Feature modules
- Interface modules (header, navigation, buttons, tables)
- Code modules (libraries, services, dependencies)

**Interpayshares aims to connect disparate models of modular systems into one holistic, expressive, interconnected system.**

#### Interpayshares has several technical design goals. It should enable:
- Development of different apps from reusable, modular pieces of the same system
- Development of features without codependencies (i.e., no house-of-cards style collapsing if you decide to pull out one feature)
- On-the-fly tests of beta features without impact on existing features
- Easy third-party customization

## Quick overview

![overview](https://www.payshares.org/wp-content/uploads/2015/06/interpayshares-overview.png)

## Prerequisites

Before you start make sure you have [Node](https://nodejs.org/) v0.10 installed (use [nvm](https://github.com/creationix/nvm) if you don't) and [npm](https://npmjs.com).

Interpayshares is written in JavaScript. We're using ECMAScript 6 standard thanks to [Babel](https://babeljs.io/) and [Webpack](http://webpack.github.io/) for bundling. Knowledge of [AngularJS](https://angularjs.org/) will be very helpful.

Please install [Yeoman](http://yeoman.io/) to quickly generate sample project in this tutorial:
```bash
npm install -g yo generator-interpayshares
```

## Installing `interpayshares` command line tool

[`interpayshares` command line tool](https://github.com/payshares/interpayshares) will help you build and develop your projects. To install `interpayshares` run:

```bash
npm install -g interpayshares
```

Make sure it's successfully installed by running `interpayshares stroopy`. You may want to check out [Stroopy adventures](https://www.payshares.org/stories/adventures-in-galactic-consensus-chapter-1/) too :).

## Generating sample app

Let's start with bootstrapping our application using `yo`:
```bash
mkdir sample-app
cd sample-app
yo interpayshares sample-app
```

Now, simply run: `interpayshares develop` and a browser window with a sample app should open.

## Interpayshares application architecture

Let's look at our sample app code and discuss the most important parts. If you have no experience with ECMAScript 6 code, please read about [ES6 features](https://github.com/lukehoban/es6features).

### Modules

Open `main.es6` file. In the top of the file you will notice `import` statements:
```js
import interpaysharesCore, {App, Inject, Intent} from "interpayshares-core";
import interpaysharesSessions from "interpayshares-sessions";
import interpaysharesNetwork from "interpayshares-network";
import interpaysharesNetworkWidgets from "interpayshares-network-widgets";
```
The most important concept of Modular Client System is modularity. Right now, there are several modules that provides different kinds of functionality for developers and [`interpayshares-network`](https://github.com/payshares/interpayshares-network) and [`interpayshares-sessions`](https://github.com/payshares/interpayshares-sessions) are two of them.

Modules can:
* Export helpful classes like `App`, `Inject` and `Intent` in the example above.
* Create angular providers, services, filter and directives that can be used by other modules and apps. `interpayshares-core` adds `interpayshares-core.Config` service that allows to read configuration variables by other modules.
* Create widgets that can be used by application developers. You can read more about widgets below.

### App configuration

As mentioned above `interpayshares-core` provides some interesting classes and `App` is one of them. `App` instance represents a single Interpayshares application. To create an app we need to pass two parameters to a constructor: application name and application config:

```js
let config = require('./config.json');
export const app = new App("sample-app", config);
```

Conceptually, an application's configuration presents itself as a simple nested json object. The interesting part, however, is that you can not only configure your application but also modules you included. When no modules configuration is present in you app config a module sets its default values. However, sometimes you may want to change it, for example [horizon](https://github.com/payshares/go-horizon) server hostname. To do it, simply add `modules.{module-name}` field in your config json and extend it with your custom variables. For example:

```json
{
  "nested": {
    "value": true
  },
  "modules": {
    "interpayshares-network": {
      "horizon": {
        "server"
      }
    }
  }
}
```

To get a configuration variable you can use `interpayshares-core.Config` service.

```js
let value = Config.get('nested.value');
console.log(value) // true
```

### App bootstrap

Every module that will be used by an app must be registered with `use` method, like:

```js
app.use(interpaysharesCore);
app.use(interpaysharesSessions);
app.use(interpaysharesNetwork);
app.use(interpaysharesNetworkWidgets);
```

This will make all Angular parts of a module (providers, services etc.) accessible from your app.

An application can contain its own Angular elements like controllers, services, templates etc. They are autoloaded using following code:

```js
app.controllers = require.context("./controllers", true);
app.templates = require.context("raw!./templates", true);
```

We're using Webpack [require.context](http://webpack.github.io/docs/context.html) to load many files at the same time.

We can configure app's [ui-router](https://github.com/angular-ui/ui-router):
```js
app.routes = ($stateProvider) => {
  $stateProvider.state('index', {
    url: "/",
    templateUrl: "sample-app/index"
  });

  $stateProvider.state('balance', {
    url: "/balance",
    templateUrl: "sample-app/balance"
  });
};
```

And finally, we can add Angular `config` and `run` blocks.

After adding all elements we can bootstrap the app using:

```js
app.bootstrap();
```

### Intents

Modules (and apps) in Interpayshares communicate by broadcasting `Intent` objects using an [Android-inspired](http://developer.android.com/guide/components/intents-filters.html) intent system. Modules can:
* **Broadcast Intents** to trigger some events in other modules,
* **Register Broadcast Receivers** to listen to Intents sent by other modules.

Every Intent has a type which must be one of standard intent types. For a complete list of Intent types please check [`interpayshares-core` documentation](https://github.com/payshares/interpayshares-core#intent-class).

Broadcast Receivers should be registered in `run` phase:

```js
let registerBroadcastReceivers = ($state, IntentBroadcast) => {
  IntentBroadcast.registerReceiver(Intent.TYPES.SHOW_DASHBOARD, intent => {
    $state.go('balance');
  });
};
registerBroadcastReceivers.$inject = ["$state", "interpayshares-core.IntentBroadcast"];
app.run(registerBroadcastReceivers);
```

Intents are sent using `interpayshares-core.IntentBroadcast` service's `sendBroadcast` method. Let's analyze how a controller can broadcast an Intent, open: `index.controller.es6` file:
```js
this.IntentBroadcast.sendBroadcast(
  new Intent(
    Intent.TYPES.SHOW_DASHBOARD
  )
);
```
`SHOW_DASHBOARD` intent tells all receivers that user should see your application dashboard. You can see that our broadcast receiver changes a current state of router to our dashboard.

Intent system is the important mechanism of communication between widgets.

### Dependency Injection

Interpayshares application is the AngularJS application and it also makes use of [Dependency Injection](https://docs.angularjs.org/guide/di) design pattern which Angular implements. In Interpayshares you can use `$inject` property annotation to inject services but you can also use `@Inject` [decorator](https://github.com/wycats/javascript-decorators) provided by `interpayshares-core` module like this:

```js
import {Inject} from 'interpayshares-core';

@Inject("interpayshares-core.Config", "interpayshares-core.IntentBroadcast", "interpayshares-sessions.Sessions")
class IndexController {
  constructor(Config, IntentBroadcast, Sessions) {
    // constructor code
  }
}
```

Important thing to notice is how services names are generated. Basically, a service name is a concatenation of module name and service name with a dot in the middle. Check a table below to find out how other artifacts names are generated:

Artifact type | Artifact name | Generated name
--- | --- | ---
Controller | `FooController` | `interpayshares-module.FooController`
Directive | `fooDirective` | `interpayshares-module-foo-directive`
Service | `FooService` | `interpayshares-module.FooService`
Provider | `FooProvider` | `interpayshares-module.FooProvider`

### Widgets

The last important part of Interpayshares we will mention in this document are widgets. Widgets are complex solutions that are reponsible for a single task, like: showing account balances. They consist of templates, controllers and directives. Widgets can be used by writing a directive name in your application template:

```html
<interpayshares-network-widgets-balance></interpayshares-network-widgets-balance>
```

As with services, widget name is a concatenation of module name and service name but this time with a hyphen in the middle.

You can find full list of classes, services and widgets provided by each module in its documentation.

## Where to go from here?

<!-- this section is copied in other .md files in this docs folder -->
* Join our [Slack channel](http://slack.payshares.org/) to discuss Interpayshares and share ideas!
* Help us develop our [interpayshares-client](https://github.com/payshares/interpayshares-client).
* [Fix issues](https://github.com/issues?utf8=%E2%9C%93&q=is%3Aopen+repo%3Apayshares%2Finterpayshares+repo%3Apayshares%2Finterpayshares-core+repo%3Apayshares%2Finterpayshares-network+repo%3Apayshares%2Finterpayshares-network-widgets+repo%3Apayshares%2Finterpayshares-wallet+repo%3Apayshares%2Finterpayshares-sessions+repo%3Apayshares%2Finterpayshares-client) in Interpayshares modules.
* Develop your own module!

## Getting a Payshares testnet account
The yeoman generator generates a testnet account for you automatically, but you can also create one for yourself using our [friendbot](https://www.payshares.org/galaxy#friendbot).
