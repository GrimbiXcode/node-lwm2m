# node-lwm2m

[![build status][travis-image]][travis-url]
[![coverage status][coveralls-image]][coveralls-url]
[![code analysis][bithound-image]][bithound-url]
[![gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/node-lwm2m/Lobby)

[travis-image]: https://img.shields.io/travis/moleike/node-lwm2m/develop.svg

[travis-url]: https://travis-ci.org/moleike/node-lwm2m

[appveyor-image]: https://img.shields.io/appveyor/ci/moleike/node-lwm2m/develop.svg

[appveyor-url]: https://ci.appveyor.com/project/moleike/node-lwm2m

[coveralls-url]: https://coveralls.io/github/moleike/node-lwm2m?branch=develop

[coveralls-image]: https://img.shields.io/coveralls/moleike/node-lwm2m/develop.svg

[bithound-url]: https://www.bithound.io/github/moleike/node-lwm2m

[bithound-image]: https://www.bithound.io/github/moleike/node-lwm2m/badges/code.svg

[**node-lwm2m**][self] is an implementation of the Open Mobile Alliance's Lightweight M2M protocol (LWM2M). 

### What is LWM2M?

LWM2M is a profile for device services based on CoAP ([RFC 7252][coap]). LWM2M defines a simple object model and a number of interfaces and operations for device management. See an overview of the protocol [here][lwm2m].

[self]: https://github.com/moleike/node-lwm2m.git

[coap]: https://tools.ietf.org/html/rfc7252

[lwm2m]: http://www.openmobilealliance.org/wp/overviews/lightweightm2m_overview.html

### Object Model

The OMA LWM2M object model is based on a simple 2 level class hierarchy consisting of *Objects* and *Resources*:

* A *Resource* is a REST endpoint, allowed to be a single value or an array of values of the same data type.

* An *Object* is a resource template and container type that encapsulates a set of related resources.  An LWM2M Object represents a specific type of information source; for example, there is a LWM2M Device Management object that represents a network connection, containing resources that represent individual properties like radio signal strength.

Source: <https://tools.ietf.org/html/draft-ietf-core-resource-directory>

### Object Registry

See [IPSO Registry][ipso] or [OMNA Registry][omna] for Object and Resource definitions.

[ipso]: https://github.com/IPSO-Alliance/pub/tree/master/reg/xml

[omna]: http://www.openmobilealliance.org/wp/OMNA/LwM2M/LwM2MRegistry.html

## Install

    npm install --save lwm2m

## Synopsis

```js
var server = require('lwm2m').createServer();

server.on('register', function(params, accept) {
  setImmediate(function() {
    server
    .read(params.ep, '3/0')
    .then(function(device) {
      console.log(JSON.stringify(device, null, 4));
    })
  });
  accept();
});

server.listen(5683);
```

## Contribute

Please report bugs via the
[github issue tracker](https://github.com/moleike/node-lwm2m/issues).

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [schemas](#schemas)
-   [createServer](#createserver)
-   [bootstrap#createServer](#bootstrapcreateserver)
-   [Resource](#resource)
-   [Schema](#schema)
    -   [validate](#validate)
-   [Server](#server)
    -   [read](#read)
    -   [write](#write)
    -   [execute](#execute)
    -   [discover](#discover)
    -   [writeAttributes](#writeattributes)
    -   [create](#create)
    -   [delete](#delete)
    -   [observe](#observe)
    -   [cancel](#cancel)
-   [bootstrap#Server](#bootstrapserver)
    -   [write](#write-1)
    -   [delete](#delete-1)
    -   [finish](#finish)
-   [Registry](#registry)
    -   [\_find](#_find)
    -   [\_get](#_get)
    -   [\_save](#_save)
    -   [\_update](#_update)
    -   [\_delete](#_delete)

### schemas

Schemas for OMA-defined objects.
See [oma](lib/oma).

### createServer

Returns **[Server](#server)** object

### bootstrap#createServer

Returns **[bootstrap#Server](#bootstrapserver)** object

### Resource

Schema resource type definition

Type: [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)

**Properties**

-   `type` **(`"String"` \| `"Integer"` \| `"Float"` \| `"Boolean"` \| `"Opaque"` \| `"Time"` | \[`"type"`])** 
-   `id` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** Resource ID
-   `required` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** resource is mandatory. Defaults to `false`
-   `enum` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)?** 
-   `range` **[object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
    -   `range.min` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
    -   `range.max` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** 

### Schema

Schema constructor.

An `Schema` describes the shape of an LwM2M Object and the type of its resources. 
Schemas are used throghout the API for generating/parsing payloads from/to JavaScript values.

See [oma](lib/oma) directory for default definitions.
See also [thermostat.js](examples/thermostat.js) for an
example of a composite schema.

**Note**

_LwM2M types will be coerced to JavaScript types and viceversa, e.g. `Time` will return a `Date()`,
`Opaque` a `Buffer()`, and `Integer`/`Float` a number._

**Parameters**

-   `resources` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String), [Resource](#resource)>** 

**Examples**

```javascript
// IPSO light controller
var lightControlSchema = new Schema({
  onOff: {
    type: 'Boolean',
    id : 5850,
    required: true
  },
  dimmer: {
    type: 'Integer',
    id: 5851,
    range: { min: 0, max: 100 }
  },
  units: {
    type: 'String',
    id: 5701,
  }
});

// an object literal matching the schema above
var lightControl = {
  onOff: true,
  dimmer: 40,
}

// Bad schema
var schema = new Schema({
  a: { type: 'String', id: 0 },
  b: { type: 'Error', id: 1 },
}); // throws TypeError
```

-   Throws **any** Will throw an error if fails to validate

#### validate

validates `obj` with `schema`.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 

**Examples**

```javascript
var schema = new Schema({
  a: { type: String, id: 0 },
  b: { type: Buffer, id: 1 },
});

schema.validate({
  a: 'foo',
  b: Buffer.from('bar'),
}); // OK

schema.validate({
  a: 'foo',
  b: 'bar',
}); // Throws error
```

-   Throws **any** Will throw an error if fails to validate

### Server

**Extends EventEmitter**

Server constructor.

Events:

-   `register`: device registration request.
-   `update`: device registration update.
-   `unregister`: device unregistration.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
    -   `options.type` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** IPv4 (udp4) or IPv6 (udp6) connections (optional, default `'upd6'`)
    -   `options.piggybackReplyMs` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** milliseconds to wait for a piggyback response (optional, default `50`)
    -   `options.registry` **[Registry](#registry)** impl. of CoRE Resource Directory (optional, default `Registry`)

#### read

Read `path` on device with endpoint name `endpoint`. The optional callback is given
the two arguments `(err, res)`, where `res` is parsed using `schema`.

Note:

_If no schema is provided will return a `Buffer` if the payload is `TLV`-encoded
or opaque, or an `String` otherwise._

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** either an LWM2M Object instance or resource
-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
    -   `options.format` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** media type. (optional, default `'text'`)
    -   `options.schema` **[Schema](#schema)** defining resources.
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

**Examples**

```javascript
var schema = Schema({
  test: { id: 1, type: Number }
});

var options = { 
  schema: schema, 
  format: 'json',
};

server.read('test', '/1024/11', options, function(err, res) {
  assert(res.hasOwnProperty('test'));
  assert(typeof res.test == 'number');
});
```

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;([Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Buffer](https://nodejs.org/api/buffer.html) \| [number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number))>** a promise of the eventual value

#### write

Write `value` into `path` of device with endpoint name `endpoint`.
For writing Object Instances, an schema is required.

Note:

_schemas can be globally added to `lwm2m.schemas`._

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `value` **([Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) \| [Buffer](https://nodejs.org/api/buffer.html))** 
-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `options.format` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** media type. (optional, default `'tlv'`)
    -   `options.schema` **[Schema](#schema)?** schema to serialize value.
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

**Examples**

```javascript
var schema = Schema({
  foo : { 
    id: 5, 
    type: 'String' 
  },
  bar : { 
    id: 6, 
    type: 'Number' 
  },
});

var options = { 
  schema: schema, 
  format: 'json',
};

var value = {
  foo: 'test',
  bar: 42,
};

var promise = server.write('test', '/42/0', value, options)
var promise = server.write('test', '/42/0/5', 'test')
var promise = server.write('test', '/42/0/6', 42)

// add schema for Object ID 42 globally.
lwm2m.schemas[42] = schema;

var promise = server.write('test', '/42/0', value)
```

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### execute

Makes an Execute operation over the designed resource ID of the selected device.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `value` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### discover

Execute a discover operation for the selected resource.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>** a promise with an strng in link-format

#### writeAttributes

Write `attributes` into `path` of endpoint `endpoint`.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `attributes` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

**Examples**

```javascript
var attr = {
  "pmin": 5,
  "pmax": 10
};

server.writeAttributes('dev0', '3303/0/5700', attr, function(err, res) {
   assert.ifError(err);
});
```

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### create

Create a new LWM2M Object for `path`, where path is an Object ID.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `value` **([Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) \| [Buffer](https://nodejs.org/api/buffer.html))** 
-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### delete

Deletes the LWM2M Object instance in `path` of endpoint `endpoint`

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### observe

Observe changes in `path` of device with endpoint name `endpoint`. 
The notification behaviour, e.g. periodic or event-triggered reporting, is configured with the 
`writeAttributes` method. The callback is given the two arguments `(err, stream)`, 
where `stream` is a `Readable Stream`. To stop receiving notifications `close()` the stream
and (optionally) call `cancel()` on the same `endpoint` and `path` and .

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `options`  
    -   `options.format` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** media type. (optional, default `'text'`)
    -   `options.schema` **[Schema](#schema)** defining resources.
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

**Examples**

```javascript
server.observe('dev0', '/1024/10/1', function(err, stream) {
  stream.on('data', function(value) {
    console.log('new value %s', value);
  });

  stream.on('end', function() {
    console.log('stopped observing');
  });
});
```

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### cancel

Cancel an observation for `path` of device `endpoint`.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

### bootstrap#Server

**Extends EventEmitter**

Server constructor.

Events

-   `bootstrapRequest`: device bootstrap request.

**Parameters**

-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
    -   `options.type` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** IPv4 (udp4) or IPv6 (udp6) connections (optional, default `'upd6'`)
    -   `options.piggybackReplyMs` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** milliseconds to wait for a piggyback response (optional, default `50`)

**Examples**

```javascript
var bootstrap = require('lwm2m').bootstrap;
var server = bootstrap.createServer();

server.on('error', function(err) {
  throw err;
});

server.on('close', function() {
  console.log('server is done');
});

server.on('bootstrapRequest', function(params, accept) {
  console.log('endpoint %s contains %s', params.ep, params.payload);
  accept();
});

// the default CoAP port is 5683
server.listen();
```

#### write

Makes a Write operation over the designed resource ID of the selected device.

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `value` **([Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) \| [Buffer](https://nodejs.org/api/buffer.html))** 
-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)?** 
    -   `options.format` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** media type.
    -   `options.schema` **[Schema](#schema)** schema to serialize value when an object.
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

**Examples**

```javascript
var schema = Schema({
  foo : { 
    id: 5, 
    type: 'String' 
  },
  bar : { 
    id: 6, 
    type: 'Number' 
  },
});

var options = { 
  schema: schema, 
  format: 'json',
};

var value = {
  foo: 'test',
  bar: 42,
};

var promise = server.write('test', '/42/3', value, options)
```

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### delete

Deletes the LWM2M Object instance in `path` of endpoint `endpoint`

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `path` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

#### finish

Terminate the Bootstrap Sequence previously initiated

**Parameters**

-   `endpoint` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** client endpoint name
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)?** 

Returns **[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)** 

### Registry

**Extends EventEmitter**

Registry for clients. 
Default implementation is in-memory.

For production use, extend `Registry` class and 
give new implementations to
\_get, \_find, \_save, \_update and \_delete.

See [examples](examples) for a MongoDB-backed registry.

#### \_find

get client by endpoint name

**Parameters**

-   `endpoint` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** callback is given
    the two arguments `(err, client)`

#### \_get

get client by location in the registry

**Parameters**

-   `location` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** callback is given
    the two arguments `(err, client)`

#### \_save

store a new client in the registry

**Parameters**

-   `client` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** callback is given
    the two arguments `(err, location)`

#### \_update

update a client in the registry

**Parameters**

-   `location` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `params` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** callback is given
    the two arguments `(err, location)`

#### \_delete

delete client from the registry

**Parameters**

-   `location` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `callback` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** callback is given
    the two arguments `(err, client)`
