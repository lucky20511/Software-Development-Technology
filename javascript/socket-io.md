From: [https://github.com/socketio/socket.io](https://github.com/socketio/socket.io)

# socket.io

**Features**

Socket.IO enables real-time bidirectional event-based communication. It consists in:

* a Node.js server \(this repository\)
* a
  [Javascript client library](https://github.com/socketio/socket.io-client)
  for the browser \(or a Node.js client\)

Some implementations in other languages are also available:

* [Java](https://github.com/socketio/socket.io-client-java)
* [C++](https://github.com/socketio/socket.io-client-cpp)
* [Swift](https://github.com/socketio/socket.io-client-swift)

Its main features are:

#### Reliability

Connections are established even in the presence of:

* proxies and load balancers.
* personal firewall and antivirus software.

For this purpose, it relies on[Engine.IO](https://github.com/socketio/engine.io), which first establishes a long-polling connection, then tries to upgrade to better transports that are "tested" on the side, like WebSocket. Please see the[Goals](https://github.com/socketio/engine.io#goals)section for more information.

#### Auto-reconnection support

Unless instructed otherwise a disconnected client will try to reconnect forever, until the server is available again. Please see the available reconnection options[here](https://github.com/socketio/socket.io-client/blob/master/docs/API.md#new-managerurl-options).

#### Disconnection detection

An heartbeat mechanism is implemented at the Engine.IO level, allowing both the server and the client to know when the other one is not responding anymore.

That functionality is achieved with timers set on both the server and the client, with timeout values \(the`pingInterval`and`pingTimeout`parameters\) shared during the connection handshake. Those timers require any subsequent client calls to be directed to the same server, hence the`sticky-session`requirement when using multiples nodes.

#### Binary support

Any serializable data structures can be emitted, including:

* [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
  and
  [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob)
  in the browser
* [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
  and
  [Buffer](https://nodejs.org/api/buffer.html)
  in Node.js

#### Simple and convenient API

Sample code:

```js
io.on('connection', function(socket){
  socket.emit('request', /* */); // emit an event to the socket
  io.emit('broadcast', /* */); // emit an event to all connected sockets
  socket.on('reply', function(){ /* */ }); // listen to the event
});
```

#### Cross-browser

Browser support is tested in Saucelabs:

[![](https://camo.githubusercontent.com/fface5a8523859ace9b349fc8922af0a8d6941f4/68747470733a2f2f73617563656c6162732e636f6d2f62726f777365722d6d61747269782f736f636b65742e737667 "Sauce Test Status")](https://saucelabs.com/u/socket)

#### Multiplexing support

In order to create separation of concerns within your application \(for example per module, or based on permissions\), Socket.IO allows you to create several`Namespaces`, which will act as separate communication channels but will share the same underlying connection.

#### Room support

Within each`Namespace`, you can define arbitrary channels, called`Rooms`, that sockets can join and leave. You can then broadcast to any given room, reaching every socket that has joined it.

This is a useful feature to send notifications to a group of users, or to a given user connected on several devices for example.

**Note:**Socket.IO is not a WebSocket implementation. Although Socket.IO indeed uses WebSocket as a transport when possible, it adds some metadata to each packet: the packet type, the namespace and the ack id when a message acknowledgement is needed. That is why a WebSocket client will not be able to successfully connect to a Socket.IO server, and a Socket.IO client will not be able to connect to a WebSocket server \(like`ws://echo.websocket.org`\) either. Please see the protocol specification[here](https://github.com/socketio/socket.io-protocol).

## Installation

```js
npm install socket.io --save
```

## How to use

The following example attaches socket.io to a plain Node.JS HTTP server listening on port`3000`.

```js
var server = require('http').createServer();
var io = require('socket.io')(server);
io.on('connection', function(client){
  client.on('event', function(data){});
  client.on('disconnect', function(){});
});
server.listen(3000);
```

### Standalone

```js
var io = require('socket.io')();
io.on('connection', function(client){});
io.listen(3000);
```

### In conjunction with Express

Starting with**3.0**, express applications have become request handler functions that you pass to`http`or`httpServer`instances. You need to pass the`Server`to`socket.io`, and not the express application function. Also make sure to call`.listen`on the`server`, not the`app`.

```js
var app = require('express')();
var server = require('http').createServer(app);
var io = require('socket.io')(server);
io.on('connection', function(){ /* … */ });
server.listen(3000);
```

### In conjunction with Koa

Like Express.JS, Koa works by exposing an application as a request handler function, but only by calling the`callback`method.

```js
var app = require('koa')();
var server = require('http').createServer(app.callback());
var io = require('socket.io')(server);
io.on('connection', function(){ /* … */ });
server.listen(3000);
```

## Documentation

Please see the documentation[here](https://github.com/socketio/socket.io/blob/master/docs/README.md). Contributions are welcome!

## Debug / logging

Socket.IO is powered by[debug](https://github.com/visionmedia/debug). In order to see all the debug output, run your app with the environment variable`DEBUG`including the desired scope.

To see the output from all of Socket.IO's debugging scopes you can use:

```
DEBUG=socket.io* node myapp
```

## Testing

```
npm test
```

This runs the`gulp`task`test`. By default the test will be run with the source code in`lib`directory.

Set the environmental variable`TEST_VERSION`to`compat`to test the transpiled es5-compat version of the code.

The`gulp`task`test`will always transpile the source code into es5 and export to`dist`first before running the test.

## Backers

Support us with a monthly donation and help us continue our activities. \[[Become a backer](https://opencollective.com/socketio#backer)\]

[![](https://camo.githubusercontent.com/ce94e3feff80cb24fac98b4a5af69ea21309e1e3/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f302f6176617461722e737667)](https://opencollective.com/socketio/backer/0/website)[![](https://camo.githubusercontent.com/c8172ecc55bd0b5a7665d41a9fdb92f44c469d30/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f312f6176617461722e737667)](https://opencollective.com/socketio/backer/1/website)[![](https://camo.githubusercontent.com/37954b978f7027bb570963fc800f404ab85752a2/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f322f6176617461722e737667)](https://opencollective.com/socketio/backer/2/website)[![](https://camo.githubusercontent.com/f90688936d18f9cf5d65191bb6310dbbbfe38199/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f332f6176617461722e737667)](https://opencollective.com/socketio/backer/3/website)[![](https://camo.githubusercontent.com/30fc7e3cbe056b21657afe5ea2848a6e24c06ff1/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f342f6176617461722e737667)](https://opencollective.com/socketio/backer/4/website)[![](https://camo.githubusercontent.com/294e9e0780d1cc30287a3b5f77483d2c0537f97a/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f352f6176617461722e737667)](https://opencollective.com/socketio/backer/5/website)[![](https://camo.githubusercontent.com/64e65608d499e02746f8638ae67da8954fa23198/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f362f6176617461722e737667)](https://opencollective.com/socketio/backer/6/website)[![](https://camo.githubusercontent.com/dd5bd148cf9af8a0902acfec8d631dd8cc2aa30f/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f372f6176617461722e737667)](https://opencollective.com/socketio/backer/7/website)[![](https://camo.githubusercontent.com/980028e84113f0fe08352f803423674636ceef61/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f382f6176617461722e737667)](https://opencollective.com/socketio/backer/8/website)[![](https://camo.githubusercontent.com/9ad65c92f3e13ee1f324dae1d9d10755a79847c9/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f392f6176617461722e737667)](https://opencollective.com/socketio/backer/9/website)[![](https://camo.githubusercontent.com/b94997d1d2ba14ad3a3edad42ed01d1a95183de9/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31302f6176617461722e737667)](https://opencollective.com/socketio/backer/10/website)[![](https://camo.githubusercontent.com/bc5a71893af6860ce1f9c2087625036c51446896/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31312f6176617461722e737667)](https://opencollective.com/socketio/backer/11/website)[![](https://camo.githubusercontent.com/3dc005b2eeb425429ddd6215c6c7320471076e1e/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31322f6176617461722e737667)](https://opencollective.com/socketio/backer/12/website)[![](https://camo.githubusercontent.com/83049e1bc2998fd7290cf1d4e5c86ececaad69fb/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31332f6176617461722e737667)](https://opencollective.com/socketio/backer/13/website)[![](https://camo.githubusercontent.com/d50bfc178e67aa476981317f9f9efa4bdcc0f859/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31342f6176617461722e737667)](https://opencollective.com/socketio/backer/14/website)[![](https://camo.githubusercontent.com/6597767b98a18bf725d4abfc60b50452725621f3/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31352f6176617461722e737667)](https://opencollective.com/socketio/backer/15/website)[![](https://camo.githubusercontent.com/6b3b415bfe31b9fa4b87282963f2c908f9d600d5/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31362f6176617461722e737667)](https://opencollective.com/socketio/backer/16/website)[![](https://camo.githubusercontent.com/8c03ef59d08092291347a33a7e7e70b6dae04ea7/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31372f6176617461722e737667)](https://opencollective.com/socketio/backer/17/website)[![](https://camo.githubusercontent.com/90fecfa540fbf1e3b96a9007f7ece9959a17cfbc/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31382f6176617461722e737667)](https://opencollective.com/socketio/backer/18/website)[![](https://camo.githubusercontent.com/ac1c92fff991569e7e7352804f537f264e3656e0/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f31392f6176617461722e737667)](https://opencollective.com/socketio/backer/19/website)[![](https://camo.githubusercontent.com/16b0e5f7a69c82f415b1cc630eb5606cbebba99e/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32302f6176617461722e737667)](https://opencollective.com/socketio/backer/20/website)[![](https://camo.githubusercontent.com/99eeed229dce358a2ed8bbbb6a030600dc5c8329/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32312f6176617461722e737667)](https://opencollective.com/socketio/backer/21/website)[![](https://camo.githubusercontent.com/0f61bd27ce8eff3b9b710fae7185f7998bb6db6a/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32322f6176617461722e737667)](https://opencollective.com/socketio/backer/22/website)[![](https://camo.githubusercontent.com/55ec4b7da38b5a00a0c27b29fe0cbd27891ea03f/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32332f6176617461722e737667)](https://opencollective.com/socketio/backer/23/website)[![](https://camo.githubusercontent.com/7d2d5fc003bc2ea95f56bdc4e1249dd981b53aae/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32342f6176617461722e737667)](https://opencollective.com/socketio/backer/24/website)[![](https://camo.githubusercontent.com/6644c32341756c3f156f12e9f55ac76082547a73/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32352f6176617461722e737667)](https://opencollective.com/socketio/backer/25/website)[![](https://camo.githubusercontent.com/2ca8e1f78012847dc7a8120e541f1fc099141722/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32362f6176617461722e737667)](https://opencollective.com/socketio/backer/26/website)[![](https://camo.githubusercontent.com/3295289d93f1fc10d9321388b282928b8579350b/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32372f6176617461722e737667)](https://opencollective.com/socketio/backer/27/website)[![](https://camo.githubusercontent.com/c1d793a30ea49c9e4b3399f72534adb8731f9286/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32382f6176617461722e737667)](https://opencollective.com/socketio/backer/28/website)[![](https://camo.githubusercontent.com/a084b98097bfb6b300b4f10fa4d0c5ad0a8c2e64/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f6261636b65722f32392f6176617461722e737667)](https://opencollective.com/socketio/backer/29/website)

## Sponsors

Become a sponsor and get your logo on our README on Github with a link to your site. \[[Become a sponsor](https://opencollective.com/socketio#sponsor)\]

[![](https://camo.githubusercontent.com/b5c30f22bb3d8c473be42915081577a0159bc0be/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f302f6176617461722e737667)](https://opencollective.com/socketio/sponsor/0/website)[![](https://camo.githubusercontent.com/80fcf5a5854fd26577ffdea96f0682728a918350/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f312f6176617461722e737667)](https://opencollective.com/socketio/sponsor/1/website)[![](https://camo.githubusercontent.com/1736081eba1985c0541a7d495a947813cc93809b/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f322f6176617461722e737667)](https://opencollective.com/socketio/sponsor/2/website)[![](https://camo.githubusercontent.com/6a4a1902771b3000e5f8b74d47ad4c3aee0fad78/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f332f6176617461722e737667)](https://opencollective.com/socketio/sponsor/3/website)[![](https://camo.githubusercontent.com/079a90ae6a083f36dab6abf66b973fcb91424c3b/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f342f6176617461722e737667)](https://opencollective.com/socketio/sponsor/4/website)[![](https://camo.githubusercontent.com/95fe5d595e7c7d420cb52aea9e5f0e2d7521caaa/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f352f6176617461722e737667)](https://opencollective.com/socketio/sponsor/5/website)[![](https://camo.githubusercontent.com/d8dd6e07c86f89ee9b285ef1a2d64ff5160ea7ee/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f362f6176617461722e737667)](https://opencollective.com/socketio/sponsor/6/website)[![](https://camo.githubusercontent.com/8e59ecc8aef68c944a01974777daabc5011fb9dc/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f372f6176617461722e737667)](https://opencollective.com/socketio/sponsor/7/website)[![](https://camo.githubusercontent.com/4ca2584f2af1eb0fee5fcaa23864db1ec98729ba/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f382f6176617461722e737667)](https://opencollective.com/socketio/sponsor/8/website)[![](https://camo.githubusercontent.com/673fa592be13950654fa9d633a75c41894986f8e/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f392f6176617461722e737667)](https://opencollective.com/socketio/sponsor/9/website)[![](https://camo.githubusercontent.com/b38e80ad950de435756fc48cecbc806fe1e69800/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31302f6176617461722e737667)](https://opencollective.com/socketio/sponsor/10/website)[![](https://camo.githubusercontent.com/c360a8a939e70f0bc00013f679521ef6e44c1ca9/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31312f6176617461722e737667)](https://opencollective.com/socketio/sponsor/11/website)[![](https://camo.githubusercontent.com/d83a90b873f9d7824372f070eeb092ee6191b437/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31322f6176617461722e737667)](https://opencollective.com/socketio/sponsor/12/website)[![](https://camo.githubusercontent.com/2fa64336c33108227332e2f3616a4632b1b96b86/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31332f6176617461722e737667)](https://opencollective.com/socketio/sponsor/13/website)[![](https://camo.githubusercontent.com/fe31b8e36b3b51cf915050faeaaab2b22146b0f8/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31342f6176617461722e737667)](https://opencollective.com/socketio/sponsor/14/website)[![](https://camo.githubusercontent.com/25801d3f5bb55593ab052031b59c4b444ff0f32d/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31352f6176617461722e737667)](https://opencollective.com/socketio/sponsor/15/website)[![](https://camo.githubusercontent.com/09dbd115caa9142bcdd0ebfedf9bccda83177427/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31362f6176617461722e737667)](https://opencollective.com/socketio/sponsor/16/website)[![](https://camo.githubusercontent.com/1c286231dbfe234790b7303e2d448e06594fb999/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31372f6176617461722e737667)](https://opencollective.com/socketio/sponsor/17/website)[![](https://camo.githubusercontent.com/b46be6dbd696eb2d3d63cd854ec9d72d1ff2d549/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31382f6176617461722e737667)](https://opencollective.com/socketio/sponsor/18/website)[![](https://camo.githubusercontent.com/0ccb3f89120c68752fabda1ba934103eaee192cd/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f31392f6176617461722e737667)](https://opencollective.com/socketio/sponsor/19/website)[![](https://camo.githubusercontent.com/33a137c1c0b19c341fec3853bebae10be7088b20/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32302f6176617461722e737667)](https://opencollective.com/socketio/sponsor/20/website)[![](https://camo.githubusercontent.com/4fe4d4307f09f79b1dadfb4ff6cc6140f4189c70/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32312f6176617461722e737667)](https://opencollective.com/socketio/sponsor/21/website)[![](https://camo.githubusercontent.com/06ccdb3a07467316fa00eb0eb3eaf37369e28075/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32322f6176617461722e737667)](https://opencollective.com/socketio/sponsor/22/website)[![](https://camo.githubusercontent.com/0925fad6fc4e04d177f091cb3be9179272b0dace/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32332f6176617461722e737667)](https://opencollective.com/socketio/sponsor/23/website)[![](https://camo.githubusercontent.com/b0c682189d48dec2fe79a68c28a292299bc0f83b/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32342f6176617461722e737667)](https://opencollective.com/socketio/sponsor/24/website)[![](https://camo.githubusercontent.com/89c5d8a1a7bd93052cc88ca37cbc179c046d333d/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32352f6176617461722e737667)](https://opencollective.com/socketio/sponsor/25/website)[![](https://camo.githubusercontent.com/e30488c8dc4c318edfac2e82f3f1600be5f9ec49/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32362f6176617461722e737667)](https://opencollective.com/socketio/sponsor/26/website)[![](https://camo.githubusercontent.com/62d5c1b6ff08fe2ee82469f1397ec9e172152aff/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32372f6176617461722e737667)](https://opencollective.com/socketio/sponsor/27/website)[![](https://camo.githubusercontent.com/1804be67f2d2d6af70295dd8ad7c26bdae5b7422/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32382f6176617461722e737667)](https://opencollective.com/socketio/sponsor/28/website)[![](https://camo.githubusercontent.com/5b25f18251d8b8527819740fd54af6344d41068f/68747470733a2f2f6f70656e636f6c6c6563746976652e636f6d2f736f636b6574696f2f73706f6e736f722f32392f6176617461722e737667)](https://opencollective.com/socketio/sponsor/29/website)

## License

[MIT](https://github.com/socketio/socket.io/blob/master/LICENSE)

