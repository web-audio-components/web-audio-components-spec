
# Web Audio Component Spec

This document defines the requirements that an audio component must meet in order
to be compliant with the Web Audio Component service.

## Component(1)

First and foremost, Web Audio Components are built with [Component(1)](https://github.com/component/component). If you haven't already done so, it is recommended that you
familiarize yourself with the Component(1) spec and workflow. 

## Constructor

A Web Audio Component **MUST** export only a single constructor whose instances support all of the property requirements below. Constructors exposed via `module.exports` **MUST** have the following signature

```javascript
ExampleModule(context, options)
```

- `context` A Web Audio Component constructor **MUST** take an [AudioContext](http://www.w3.org/TR/webaudio/#AudioContext-section) as the first argument. 
- `options` The constructor **MAY** take an **OPTIONAL** second argument, which, if present, **MUST** be an object containing any necessary configuration information. It is expected that properties defined in the **options** object are also public properties on the module instance itself.

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

Every module instance **MUST** have the following properties,

- `input` **MUST** be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will accept incoming connections from other AudioNodes.
- `output` **MUST** be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will make all outgoing connections.
- `meta` **MUST** be an object containing all necessary metadata related to your module. It is expected that the `.meta` object contains a property `name`, which represents the display name of your module, and a property `params`, which represents each of the public, configurable properties on your module and the minimum, maximum, and default value that property should take. Finally, a `type` property is included which can be one of `float`, `int`, or `bool` to help the graphic user interface apply the correct values.

### Demonstration of required metadata

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

Every module instance **MUST** contain `connect` and `disconnect` methods. These methods **SHOULD** delegate to the `output` node\'s native `connect` and `disconnect` methods, but should be aware that the `connect` method may need to connect to either a native AudioNode or another Web Audio Component.

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

The Web Audio Component manifest is the same as the [Component(1) manifest](https://github.com/component/component/wiki/Spec),
with one caveat. Web Audio Components **MUST** include a `web-audio` field in their component.json files. This property is
how the Web Audio Components service differentiates audio components from various other types of components. The `web-audio`
property **MUST** be an object, which **SHOULD** contain a `type` property. The `type` property can be any of the follow
values: `effect`, `tool`, `generator`. The `web-audio` object **COULD** contain additional information in the future.

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

## Contributing

If you have any comments, questions, or suggestions regarding the Web Audio Component spec,
please open an issue for discussion with the community.
