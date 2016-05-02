Simulates your Apache Cordova application in the browser.

# Installation

```
npm install -g cordova-simulate
```


# Usage

## API
Use `require('cordova-simulate')` to launch a simulation via the API:

```JavaScript
var simulate = require('cordova-simulate');
simulate(opts);
```

Where opts is an object with the following properties (all optional):

* **platform** - any Cordova platform that has been added to your project. Defaults to `browser`.
* **target** - the name of the browser to launch your app in. Can be any of the following: `chrome`, `chromium`, `edge`, `firefox`, `ie`, `opera`, `safari`. Defaults to `chrome`.
* **port** - the desired port for the server to use. Defaults to `8000`.
* **dir** - the directory to launch from (where it should look for a Cordova project). Defaults to cwd.
* **simhostui** - the directory containing the UI specific files of the simulation host. Defaults to the bundled simulation host files, found in `src/sim-host`.


# What it does

Calling `simulate()` will launch your app in the browser, and open a second browser window displaying UI (the simulation host) that allows you to configure how plugins in your app respond.

## Features

* Allows the user to configure plugin simulation through a UI. 
* Launches the application in a separate browser window so that it's not launched within an iFrame, to ease up debugging.
* Allows user to persist the settings for a plug-in response. 
* Allows plugins to customize their own UI.

## Supported plugins

This preview version currently includes built-in support for the following Cordova plugins:

* [cordova-plugin-camera](https://github.com/apache/cordova-plugin-camera)
* [cordova-plugin-console](https://github.com/apache/cordova-plugin-console)
* [cordova-plugin-contacts](https://github.com/apache/cordova-plugin-contacts)
* [cordova-plugin-device](https://github.com/apache/cordova-plugin-device)
* [cordova-plugin-device-motion](https://github.com/apache/cordova-plugin-device-motion)
* [cordova-plugin-dialogs](https://github.com/apache/cordova-plugin-dialogs)
* [cordova-plugin-file](https://github.com/apache/cordova-plugin-file)
* [cordova-plugin-geolocation](https://github.com/apache/cordova-plugin-geolocation)
* [cordova-plugin-globalization](https://github.com/apache/cordova-plugin-globalization)
* [cordova-plugin-media](https://github.com/apache/cordova-plugin-media)
* [cordova-plugin-vibration](https://github.com/apache/cordova-plugin-vibration)

## Adding simulation support to plugins

It also allows for plugins to define their own UI. To add simulation support to a plugin, follow these steps:

1. Clone this repository (`git clone https://github.com/microsoft/cordova-simulate.git`), as it contains useful example code (see `src/plugins`).
2. Add your plugin UI code to your plugin in `src/simulation`. Follow the file naming conventions seen in the built-in plugins.

### Detailed steps

In your plugin project, add a `simulation` folder under `src`, then add any of the following files:

```
sim-host-panels.html
sim-host-dialogs.html
sim-host.js
sim-host-handlers.js
app-host.js
app-host-handlers.js
app-host-clobbers.js
```
 
#### Simulation Host Files

*sim-host-panels.html*

This defines panels that will appear in the simulation host UI. At the top level, it should contain one or more
`cordova-panel` elements. The `cordova-panel` element should have an `id` which is unique to the plugin (so the plugin
name is one possibility, or the shortened version for common plugins (like just `camera` instead of
`cordova-plugin-camera`). It should also have a `caption` attribute which defines the caption of the panel.
 
The contents of the `cordova-panel` element can be regular HTML, or the various custom elements which are supported
(see the existing plugin files for more details).
 
This file shouldn't contain any JavaScript (including inline event handlers), nor should it link any JavaScript files.
Any JavaScript required can be provided in the standard JavaScript files described below, or in additional JavaScript
files that can be included using `require()`.
 
*sim-host-dialogs.html*

This defines any dialogs that will be used (dialogs are simple modal popups � such as used for the Camera plugin). At
the top level it should contain one or more `cordova-dialog` elements. Each of these must have `id` and `caption`
attributes (as for `sim-host-panels.html`). The `id` will be used in calls to `dialog.showDialog()` and
`dialog.hideDialog()` (see [cordova-simulate/src/plugins/cordova-plugin-camera/sim-host.js]
(https://github.com/Microsoft/cordova-simulate/blob/master/src/plugins/cordova-plugin-camera/sim-host.js)
for example code).
 
Other rules for this file are the same as for `sim-host-panels.html`.

*sim-host.js*

This file should contain code to initialize your UI. For example � attach event handlers, populate lists etc. It should
set `module.exports` to one of the following:
 
1. An object with an `initialize` method, like this:
 
``` js
module.exports = {
    initialize: function () {
        // Your initialization code here.
    }
};
```

2. A function that returns an object with an `initialize` method. This function will be passed a single parameter �
`messages` � which is a plugin messaging object that can be used to communicate between `sim-host` and `app-host`.
This form is used when the plugin requires that `messages` object � otherwise the simple form can be used. For example:

``` js
module.exports = function (messages) {
    return {
        initialize: function () {
            // Your initialization code here.
        }
    };
};
```

In both cases, the code *currently* executes in the context of the overall simulation host HTML document. You can use
`getElementById()` or `querySelector()` etc to reference elements in your panel to attach events etc. In the future,
this will change and there will be a well defined, limited, asynchronous API for manipulating elements in your
simulation UI.

*sim-host-handlers.js*

This file defines handlers for plugin `exec` calls. It should return an object in the following form:
 
``` js
{
    service1: {
        action1: function (success, error, args) {
            // exec handler
        },
        action2: function (success, error, args) {
            // exec handler
        }
    },
    service2: {
        action1: function (success, error, args) {
            // exec handler
        },
        action2: function (success, error, args) {
            // exec handler
        }
    }
}
```
 
It can define handlers for any number of service/action combinations. As for `sim-host.js`, it can return the object
either by;

1. Setting module.exports to this object.
2. Setting module.exports to a function that returns this object (in which case the messages parameter will be passed to that function).

#### App Host Files

*app-host.js*

This file is injected into the app itself (as part of a single, combined, `app-host.js` file). Typically, it would
contain code to respond to messages from `sim-host` code, and as such `module.exports` should be set a function that
takes a single `messages` parameter. It doesn't need to return anything.
 
*app-host-handlers.js*

This file is to provide `app-host` side handling of `exec` calls (if an `exec` call is handled on the `app-host` side,
then it doesn't need to be handled on the `sim-host` side, and in fact any `sim-host` handler will be ignored). The
format is the same as `sim-host-handlers.js`.
 
*app-host-clobbers.js*

This file provides support for "clobbering" built in JavaScript objects. It's form is similar to `app-host-handlers.js`,
expect that the returned object defines what you are clobbering. For example, the built-in support for the `geolocation`
plugin uses this to support simulating geolocation even when the plugin isn't present in the app (just like `Ripple`
does), by returning the following:

``` js
{
    navigator: {
        geolocation: {
            getCurrentPosition: function (successCallback, errorCallback, options) {
                // Blah blah blah 
            },
            watchPosition: function (successCallback, errorCallback, options) {
                // Blah blah blah
            }
        }
    }
}
```

#### The "messages" Object

A `messages` object is provided to all standard JavaScript files on both the `app-host` and `sim-host` side of things.
It provides the following methods:

`messages.call(method, param1, param2 ...)`: Calls a method implemented on "the other side" (that were registered by
calling `messages.register()`) and returns a promise for the return value, that is fulfilled when the method returns.

`messages.register(method, handler)`: Registers a method handler, which can be called via `messages.call()`.

`messages.emit(message, data)`: Emits a message with data (scalar value or JavaScript object) which will be received by
any code that registers for it (in both `app-host` and `sim-host`).

`messages.on(message, handler)`: Register interest in a particular message.

`messages.off(message, handler)`: Un-register interest in a particular message.

Note that:
* All the above methods are isolated to the plugin � that is, they can only be used to communicate within the plugin's
  own code. For example, when you emit a message, it will only be received by code for the same plugin that registers to
  hear it. So different plugins can use the same method and message names without conflict.
* A method call is always sent from `app-host` to `sim-host` or vice versa (that is, a call from `app-host` can only be
  handled by a method registered on `sim-host`, and vice versa).
* Emitted messages, on the other hand, are sent both "locally" and across to the "other side".
