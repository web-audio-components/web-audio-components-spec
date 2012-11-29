# WAPM Module Spec

## Constructor Exports

The definition **must** be a single function that may be instantiated, whose instance supports all of the property requirements below. Ultimately, the single export object should be the constructor. For module `TestModule`:

```
var testModuleNode = new TestModule( context );
testModuleNode.connect( otherNode );
```

The module **should** export via CommonJS, AMD, and bind to global object. View UMDjs (need link) for examples, or the [simple-reverb](https://github.com/wapm/simple-reverb) module for an example. Currently, the WAPM site uses requireJS/AMD to pull down the scripts for demos.

**!!!** Is there a down side to having all of these export options?

## Constructor Arguments

The module constructor **must** take in the audio context as the first argument.

The module constructor **may** take additional arguments as instantiation config, and **should** just be an options object as second argument if implemented.

```
var testModuleNode = new TestModule( context, {
  gain : 5
});
```

## Instance Properties

Each node **must** have properties of the following:

*  `context` - The audio context that all Web Audio nodes are based off of, also must be passed in as the first argument of the constructor
* `input` - Must be a Web Audio node, and can be of any type (source node, processing node, gain node, etc.). This is what other nodes connect to.
* `output` - Must be a Web Audio node, and can be of any type (source node, processing node, gain node, etc.). This is what connects to other nodes.

Note, `input` and `output` may be the same node.

## Audio Parameters

If the node has configurable audio parameters beyond instantiation, they **must** be stored as a key-value pairing object under the `params` property.

```
function customGainNode ( context ) {
  this.input = this.output = context.createGain();
  this.params = {
    gain : this.input.gain
  }
}
```

Each value in the `params` object must be either an `AudioParam`, or an object with the same interface, that is, it **must** have a `name`, `defaultValue`, `minValue`, `maxValue`, and `value`. Look at the [simple-reverb](https://github.com/wapm/simple-reverb) repo to see an example of custom audio parameters with getters and setters.

## Instance Methods

A `connect` and `disconnect` method **must** be added to the prototype. These methods should use the `output` node\'s native `connect` and `disconnect` methods, although the `connect` method needs to connect to either a native Web Audio node or one that follows these specs.

```
TestModule.prototype.connect = function ( node ) {
  this.output.connect( node && node.input ? node.input : node );
};

TestModule.prototype.disconnect = function () {
  this.output.disconnect();
};
```

## Manifest Specifications

The manifest **must** be a valid JSON file, similar to a `component.json` or `package.json`, saved as a `wapm.json`. Look at the [wapm.json](https://github.com/wapm/simple-reverb/blob/master/wapm.json) in the simple-reverb repository for an example. The following fields are **required**:

* `name` - the name of the module. Must be between 1-30 characters, alphanumeric and dashes, must start with a letter. Must be unique.
* `repo` - the repo on GitHub, in owner/repository format. For example, 'wapm/simple-reverb' refers to the repository at [https://github.com/wapm/simple-reverb](https://github.com/wapm/simple-reverb)
* `description` - a brief description of the module, used in searching
* `keywords` - an array of keywords, used in searching.
* `script` - the name of the js file that is the module, relative to the repository root.

The following attributes are **recommended**:

* `author` - a string of the author's name (TODO need to support name, url)
* `license` - a string of what license the module is released under
