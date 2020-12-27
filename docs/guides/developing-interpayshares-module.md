---
id: developing-interpayshares-module
title: Developing Interpayshares Modules
category: Getting Started
---

Developing Interpayshares Modules
===============

**Development of the `interpayshares` project has been paused. Please check out the [JS Payshares SDK](https://www.payshares.org/developers/js-payshares-sdk/learn/index.html) docs for how integrate with Payshares using JavaScript.**

This document will guide you through the process of developing your own module that will be used by the Interpayshares application.

You may want to begin with the [Getting Started](./) doc.

## Contents

1. [What is a module?](#what-is-a-module)
1. [Prerequisites](#prerequisites)
1. [Installing `interpayshares` command line tool](#installing-interpayshares-command-line-tool)
1. [Generating sample app and sample module files](#generating-sample-app-and-sample-module-files)
1. [Interpayshares module architecture](#interpayshares-module-architecture)
1. [Creating a widget](#creating-a-widget)
1. [Checking if module works](#checking-if-module-works)
1. [Where to go from here?](#where-to-go-from-here)

## What is a module?

Modules are building blocks of IMS which export services, widgets and helpful classes for developers.

Technically, a module is a NPM package and [AngularJS module](https://docs.angularjs.org/guide/module).

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

## Generating sample app and sample module files

Let's start with bootstraping our module:

```bash
mkdir interpayshares-friendbot
cd interpayshares-friendbot
yo interpayshares:module interpayshares-friendbot
npm link
cd ..
```

We're using [`npm link`](https://docs.npmjs.com/cli/link) to create a symbolic link to our sample module to be able to use a module in a sample application during development.

Now, let's bootstrap our application using `yo`:

```bash
mkdir sample-app
cd sample-app
yo interpayshares sample-app
```

We need some working application to test our module. To install our previously linked module we need to run in a new application directory:

```bash
npm link interpayshares-friendbot
```

If you're developing multiple modules in the same time you may find [`npm-workspace`](https://www.npmjs.com/package/npm-workspace) package very useful. Check our [`interpayshares-payshares-client`](https://github.com/payshares/interpayshares-payshares-client) repo to find out how we use it.

## Interpayshares module architecture

Let's look at our sample module code and discuss the most important parts. If you have no experience with ECMAScript 6 code, please read about [ES6 features](https://github.com/lukehoban/es6features).

### Module definition

Open `index.es6` file. We start by creating a new `Module` object and giving it a name. It's important to export our new module as a default value:

```js
import {Module} from "interpayshares-core";

const mod = new Module('interpayshares-friendbot');
export default mod;
```

Module can contain its own Angular elements like controllers, services, templates etc. They are autoloaded using:

```js
app.controllers = require.context("./controllers", true);
app.directives = require.context("./directives", true);
app.templates = require.context("raw!./templates", true);
```

We're using Webpack [require.context](http://webpack.github.io/docs/context.html) to load many files at a time.

### Module configuration

Modules can export a default configuration object. All of the values exposed by a module can be overwritten later by an application that's using it. To add a module configuration we use [`interpayshares-core.Config`](https://github.com/payshares/interpayshares-core#interpayshares-coreconfig-service) provider's `addModuleConfig` method.

It's important to do all of above steps in application `config` block (read more about [configuration and run blocks](https://docs.angularjs.org/guide/module)).

```js
let addConfig = ConfigProvider => {
  ConfigProvider.addModuleConfig(mod.name, {
    amount: 1000
  });
};
addConfig.$inject = ['interpayshares-core.ConfigProvider'];
mod.config(addConfig);
```

To learn more please read [`interpayshares-core.Config`](https://github.com/payshares/interpayshares-core#interpayshares-coreconfig-service) documentation.

Finally, we need to run `define` method to register module artifacts:

```js
mod.define();
```

### Module exports

Module can additionally export normal JS classes and objects. You can do it by using ES6 [`export`](https://github.com/lukehoban/es6features#modules) statement.

### Widgets

The last important part of modules we will mention in this document are widgets. Widgets are complex solutions that are reponsible for a single task, like: showing account balances. They consist of templates, controllers and directives. Widget can be embeded in an application by putting a directive name in an application template. For example:

```html
<interpayshares-network-widgets-balance></interpayshares-network-widgets-balance>
```

Widget name is a concatenation of module name and directive name with a hyphen in the middle.

## Creating a widget

In this tutorial we will develop a _friendbot_ module that will send us some [testnet](https://github.com/payshares/payshares-core/blob/master/docs/testnet.md) paysharess to play with.

To create a widget we need to create widget's controller, directive, template and styles. Let's start with a [directive](https://docs.angularjs.org/guide/directive):

```js
let friendbotWidget = function () {
  return {
    restrict: "E",
    templateUrl: "interpayshares-friendbot/friendbot-widget"
  }
};

module.exports = function(mod) {
  mod.directive("friendbot", friendbotWidget);
};
```

For devs familiar with AngularJS it's nothing new except for the last 3 lines. These lines simply register a directive in Interpayshares.

Every widget has a template. Let's open our friendbot widget template:

```html
<div ng-controller="interpayshares-friendbot.FriendbotWidgetController as widget">
  <div class="interpayshares-network-modules-receive-receiveContainer">
    <button ng-click="widget.friendbot()">Friendbot</button><span ng-if="widget.friendbotSent">Sent!</span>
  </div>
</div>
```

<!--Read more about styling your widgets [here](kelp-readme).-->

Now, let's look at a widget controller:

```js
require('../styles/friendbot-widget.scss');

@Inject("$scope", "interpayshares-core.Config", "interpayshares-sessions.Sessions", "interpayshares-network.Server")
class FriendbotWidgetController {
  constructor($scope, Config, Sessions, Server) {
    if (!Sessions.hasDefault()) {
      console.error('Active session is required by this widget.');
      return;
    }
    this.$scope = $scope;
    this.Config = Config;
    this.Server = Server;
    this.session = Sessions.default;
  }

  friendbot() {
    this.Server.friendbot(this.session.getAddress(), {
        amount: Config.get('modules.interpayshares-friendbot.amount')
      }).then(() => {
        this.$scope.$apply();
      })
  }
}

module.exports = function(mod) {
  mod.controller("FriendbotWidgetController", FriendbotWidgetController);
};

```

There are a couple of important things to mention here:

First, you should tie your widget styles to a controller by putting your `require` statement here. In the future, Interpayshares will provide conditional `require`s mechanism - when application is not using your widget, styles won't be loaded too.

Next, you can notice that modules can use other modules services. In an example above `FriendbotWidgetController` is using `interpayshares-sessions.Sessions` and `interpayshares-network.Server` services.

Another interesting part of this controller is an example of using `interpayshares-core.Config` service to read configuration value.

Finally, again we need to register controller to Interpayshares with the last 3 lines.

## Checking if module works

We're ready to check if our module works. To use our `<interpayshares-friendbot-friendbot>` widget we simply need to import our module and use it:

```js
import interpaysharesFriendbot from "interpayshares-friendbot";
app.use(interpaysharesFriendbot);
```

Then we only need to insert it somewhere in our application templates, like this:

```html
<interpayshares-friendbot-friendbot></interpayshares-friendbot-friendbot>
```

Then run `interpayshares develop` to compile and open a sample application in your browser.

## Where to go from here?

* Develop your own module!
<!-- this section is copied in other .md files in this docs folder -->
* Join our [Slack channel](http://slack.payshares.org/) to discuss Interpayshares and share ideas!
* Help us develop our [interpayshares-client](https://github.com/payshares/interpayshares-client).
* [Fix issues](https://github.com/issues?utf8=%E2%9C%93&q=is%3Aopen+repo%3Apayshares%2Finterpayshares+repo%3Apayshares%2Finterpayshares-core+repo%3Apayshares%2Finterpayshares-network+repo%3Apayshares%2Finterpayshares-network-widgets+repo%3Apayshares%2Finterpayshares-wallet+repo%3Apayshares%2Finterpayshares-sessions+repo%3Apayshares%2Finterpayshares-client) in Interpayshares modules.
* Develop your own module!
