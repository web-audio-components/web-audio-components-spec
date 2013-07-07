# Web Audio Component Spec

[discussion group](https://groups.google.com/forum/#!forum/web-audio-components)


Current version: *v0.1.0* (currently in flux, not updating)
[Past versions](https://github.com/web-audio-components/web-audio-components-spec/tags)

This document defines the requirements that a component must meet in order to be compliant and interoperable with other Web Audio Components (WACs). 

This spec is still a work in progress as we discover new uses and features. Please check out the [discussion group](https://groups.google.com/forum/#!forum/web-audio-components) to provide feedback!

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **RECOMMENDED**, **MAY** and **OPTIONAL** in this document are to be interpreted as described in [RFC2119](http://www.ietf.org/rfc/rfc2119.txt), yeah yeah yeah...

## Goals

The goals of this audio component specification are to promote interoperability and reusability of code. There are a lot of web audio demos with amazing effects and signal processing only able to be used within the demo, are bound to the global `window` object, and have differing ways of creating composite audio nodes. 

Adherence to the specification and consuming the registry allows the following:

* Web Audio Components able to connect to and be connected by other WACs and native AudioNodes
* Creation of composite audio nodes, comprised of several low-level AudioNodes, or other existing WACs
* Allowing external libraries to manipulate WACs -- an example of this can be found on [component.fm](http://component.fm) -- WACs that follow spec can be rendered with a [Rack GUI](https://github.com/web-audio-components/rack) since the components follow specification of how the parameters can be manipulated
* Allow a more succinct way of creating WACs with a library, rather than defining boilerplate for every WAC created.
* Sharing and reusing code -- rather than rewriting audio components or signal processing algorithms from scratch, just use an existing component found on the [registry](http://component.fm)

## Background

### Component(1)

First and foremost, Web Audio Components are a subset of components built with [Component(1)](https://github.com/component/component). WACs construction and manifests follow the Component(1) spec, with additional properties and requirements; if you haven't already done so, it is recommended that you familiarize yourself with the Component(1) spec and workflow.
Please note that audio components will only show up in the [Web Audio Components API](http://api.component.fm) and [registry site](http://component.fm) site after being successfully registered to the [Component(1) registry](https://github.com/component/component/wiki/Components).

### Web Audio Component Types

There are three different types of Web Audio Components currently: effect, tool, and source, explained in more detail below.

#### Effect Components

Effect components are WACs that have an input and an output, and pass a signal from input to output and **SHOULD** alter the signal in some way. Example effect components are [overdrive](http://component.fm/#components/web-audio-components/overdrive) and [delay](http://component.fm/#components/web-audio-components/delay).

#### Tool Components

**Currently in development, [follow the discussion](https://groups.google.com/forum/#!topic/web-audio-components/UqIwl-CKM3M)**
Like effect components, tool components also have an input and an output, and pass a signal from input to output. They differ from effect components as they **SHOULD NOT** alter the signal, but used to analyse the signal in some way. Example tool components are [spectrograph](https://github.com/web-audio-components/spectrograph), which draws a spectrogram of the signal passed through to a canvas element, or a component that calculates the [spectral centroid](http://en.wikipedia.org/wiki/Spectral_centroid) of a signal.

#### Source Components

**Currently in development, [follow the discussion](https://groups.google.com/forum/#!topic/web-audio-components/ZJlEFXz8PCA)**

Unlike both tool and effect components, source components *generate*, and are of some form of `AudioSourceNodes`. They have a source node, and an output.

**Further elaboration of source components is necessary, to bikeshed different use cases and `AudioSourceNodes`, and how they'd be triggered externally**

#### Synth Components

**Currently in development, [follow the discussion](https://groups.google.com/forum/#!topic/web-audio-components/x74QAYoqcrk)**

Synth components are a type of source components that accept a form of note input (either programatically or via MIDI), and provide a voice to the notes that can be chained into other native AudioNodes or WACs. Providing a similar API amongst synth components, so that swapping in and out "voices" for a track is seamless, and can route the output into an AudioContext chain, like using softsynths in a DAW.

## Manifest

Like all Component(1) components, a Web Audio Component **MUST** have a manifest in a `component.json` file, with all the properties required by Component(1). WACs **MUST** include a `web-audio` property in their `component.json` file -- this property indicates to the WAC service that this is an audio component. The `web-audio` property **MUST** be an object.

### Properties

The following properties are properties of the `web-audio` property. Additional properties may be added outside of the `web-audio` property that is supported by [component.fm](http://component.fm), such as `twitter` and `github` of the author (to display on the site), as well as `demo` for more complex audio components.

#### .type

The `web-audio` property **MAY** contain a `type` property. The `type` property can be any of the following values:

  * `effect`
  * `tool`
  * `source`
  * `synth`

The `type` may be undefined, but in which case, tools subscribing to the WAC specification may not be able to handle the audio component correctly. Types are defined in the background section above.

#### .version

The spec version the component adheres to **MUST** be listed. This should be in [semver](http://semver.org/) form as a string.

### Examples

* [web-audio-components/delay](https://github.com/web-audio-components/delay/blob/master/component.json)
* [web-audio-components/simple-reverb](https://github.com/web-audio-components/simple-reverb/blob/master/component.json)

```json
{
  "name": "overdrive",
  "repo": "web-audio-components/overdrive",
  "description": "An overdrive distortion effect for the Web Audio API.",
  "version": "0.0.4",
  "author": "Nick Thompson <ncthom91@gmail.com>",
  "twitter": "ncthomp91",
  "github": "ncthomp91",
  "keywords": [
    "web-audio",
    "effect",
    "distortion",
    "overdrive"
  ],
  "dependencies": {},
  "development": {},
  "license": "MIT",
  "demo": "http://web-audio-components.github.io/overdrive",
  "scripts": [
    "index.js"
  ],
  "web-audio": {
    "type": "effect",
    "version": "0.1.0"
  }
}
```

## Implementation

### Constructor

A Web Audio Component **MUST** export only a single constructor whose instances support all of the property requirements below. Constructors exposed via `module.exports` **MUST** have the following signature

```javascript
ExampleModule(context, options)
```

- `context` A Web Audio Component constructor **MUST** take an [AudioContext](http://www.w3.org/TR/webaudio/#AudioContext-section) as the first argument. 
- `options` The constructor **MAY** take an **OPTIONAL** second argument, which, if present, **MUST** be an object containing any necessary configuration information. In most cases, properties defined in the `options` object **SHOULD** also be public properties on the module instance itself. Exceptions to this may be a WAC that cannot be changed after instantiation, like passing in a canvas element for a tool component.

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

Every module instance **MUST** have the following properties:

- `input` **MUST** be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will accept incoming connections from other AudioNodes for tool and effect components. For source components, this is the generator node.
- `output` **MUST** be an [AudioNode](http://www.w3.org/TR/webaudio/#AudioNode-section). This node will make all outgoing connections.
- `meta` **MUST** be an object containing all necessary metadata related to your module. The `meta` object **SHOULD** contain the following properties:
  - `name` if exists, **SHOULD** represent the display name of your component.
  - `params` if exists, **MUST** be an object listing each of the public, configurable properties on the module instance. Each property of `params` **MUST** have a `defaultValue` and `type`, where `type` is one of `float`, `int`, `bool`, or `enum`. `enums` also **MUST** have a `values` array of acceptable values, and all other types **MUST** have a `min` and `max` value.

Adherence to the metadata allows other libraries to interact with WAC in a desirable, consistent way. A WAC **MAY** constrain the setters of the property to be within the allotted min and max range or an acceptable value for an enum, and also constrain to the type, but it is not necessary.

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
    },
    frequency: {
      values: [110, 220, 440, 880],
      defaultValue: 440,
      type: "enum"
    },
  }
};
```

**Note,** `input` and `output` may be the same node.

## Instance Methods

Every module instance **MUST** contain `connect` and `disconnect` methods. These methods **SHOULD** delegate to the `output` node's native `connect` and `disconnect` methods, but should be aware that the `connect` method may need to connect to either a native AudioNode or another Web Audio Component, and may require some internal rerouting upon connect or disconnect.

### Example

```javascript
ExampleModule.prototype.connect = function ( node ) {
  this.output.connect( node && node.input ? node.input : node );
};

ExampleModule.prototype.disconnect = function () {
  this.output.disconnect();
};
```

## Contributing

If you have any comments, questions, or suggestions regarding the Web Audio Component spec,
[please open an issue](https://github.com/web-audio-components/web-audio-components-spec/issues) for discussion with the community.
