
# Web Audio Component Spec

This document details the properties that an audio component must uphold in order
to be compliant with the Web Audio Component service.

## Component(1)

First and foremost, Web Audio Components are built with [Component(1)](https://github.com/component/component). If you haven't already done so, it is recommended that you
familiarize yourself with the Component(1) spec and workflow. 

## Constructor

A Web Audio Component must export only a single constructor whose instances support all of the property requirements below. Constructors exposed via `module.exports` must have the following signature

```javascript
ExampleModule(context, options)
```

- **context** A Web Audio Component constructor must take an [AudioContext](http://www.w3.org/TR/webaudio/#AudioContext-section) as the first argument. 
- **options** The constructor may take an *optional* second argument, which must be an object containing any necessary configuration information.

### Example

```javascript
var context = new webkitAudioContext()
  , ExampleModule = require("nick-thompson/examplemodule")
  , example = new ExampleModule(context, {
      gain: 0.5,
      frequency: 440
    });
```

## Instance Properties

Every module instance must have the following properties,

- `.input` Must be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will accept incoming connections from other AudioNodes.
- `.output` Must be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will make all outgoing connections.
- `.meta` Must be an object containing all necessary metadata related to your module. It is expected that the `.meta` object contains a property `name`, which represents the display name of your module, and a property `params`, which represents each of the public, configurable properties on your module and the minimum, maximum, and default value that property should take. Finally, a `type` property is included which can be one of `float`, `int`, or `boolean` to help the graphic user interface apply the correct values.

### Demonstration of required metadata.

```javascript
ExampleModule.prototype.meta = {
  name: "Example Module",
  params: {
    gain: {
      min: 0,
      max: 1,
      defaultValue: 0.5,
      type: "float"
    }
  }
};
```

**Note,** `input` and `output` may be the same node.

## Instance Methods

Every module instance must contain `connect` and `disconnect` methods. These methods should delegate to the `.output` node\'s native `connect` and `disconnect` methods, but should be aware that the `connect` method may need to connect to either a native AudioNode or another Web Audio Component.

### Example

```javascript
ExampleModule.prototype.connect = function ( node ) {
  this.output.connect( node && node.input ? node.input : node );
};

ExampleModule.prototype.disconnect = function () {
  this.output.disconnect();
};
```

## Component.json

The Web Audio Component manifest is mostly the same as the [Component(1) manifest](https://github.com/component/component/wiki/Spec). The primary difference is that, to distinguish a Web Audio Component from every other component, the additional field `web-audio` should be applied to the component.json file. The `web-audio` property must be an object containing a single key-value pair `type`, with one of `effect`, `tool`, or `generator`.

### Example

```json
{
  "name": "overdrive",
  "repo": "web-audio-components/overdrive",
  "description": "An overdrive distortion effect for the Web Audio API.",
  "version": "0.0.4",
  "author": "Nick Thompson <ncthom91@gmail.com>",
  "keywords": [
    "web-audio",
    "effect",
    "distortion",
    "overdrive"
  ],
  "dependencies": {},
  "development": {},
  "license": "MIT",
  "scripts": [
    "index.js"
  ],
  "web-audio": {
    "type": "effect"
  }
}
```
