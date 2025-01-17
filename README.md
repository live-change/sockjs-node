# This fork

All http headers are readable.

# SockJS-node

[![npm version](https://img.shields.io/npm/v/sockjs.svg?style=flat-square)](https://www.npmjs.com/package/sockjs)[![Dependencies](https://img.shields.io/david/sockjs/sockjs-node.svg?style=flat-square)](https://david-dm.org/sockjs/sockjs-node)

# SockJS for enterprise

Available as part of the Tidelift Subscription.

The maintainers of SockJS and thousands of other packages are working with Tidelift to deliver commercial support and maintenance for the open source dependencies you use to build your applications. Save time, reduce risk, and improve code health, while paying the maintainers of the exact dependencies you use. [Learn more.](https://tidelift.com/subscription/pkg/npm-sockjs?utm_source=npm-sockjs&utm_medium=referral&utm_campaign=enterprise&utm_term=repo)

# SockJS family

  * [SockJS-client](https://github.com/sockjs/sockjs-client) JavaScript client library
  * [SockJS-node](https://github.com/sockjs/sockjs-node) Node.js server
  * [SockJS-erlang](https://github.com/sockjs/sockjs-erlang) Erlang server
  * [SockJS-tornado](https://github.com/MrJoes/sockjs-tornado) Python/Tornado server
  * [vert.x](https://github.com/eclipse/vert.x) Java/vert.x server

Work in progress:

  * [SockJS-ruby](https://github.com/nyarly/sockjs-ruby)
  * [SockJS-netty](https://github.com/cgbystrom/sockjs-netty)
  * [SockJS-gevent](https://github.com/sdiehl/sockjs-gevent) ([and a fork](https://github.com/njoyce/sockjs-gevent))
  * [pyramid-SockJS](https://github.com/fafhrd91/pyramid_sockjs)
  * [wildcloud-websockets](https://github.com/wildcloud/wildcloud-websockets)
  * [SockJS-cyclone](https://github.com/flaviogrossi/sockjs-cyclone)
  * [SockJS-twisted](https://github.com/Fugiman/sockjs-twisted/)
  * [wai-SockJS](https://github.com/Palmik/wai-sockjs)
  * [SockJS-perl](https://github.com/vti/sockjs-perl)
  * [SockJS-go](https://github.com/igm/sockjs-go/)
  * [actix/sockjs](https://github.com/actix/sockjs) for Rust

⚠️️ **ATTENTION** This is pre-release documentation. The documentation for the
latest stable release is at: https://github.com/sockjs/sockjs-node/tree/v0.3.19 ️⚠️

# What is SockJS?

SockJS is a JavaScript library (for browsers) that provides a WebSocket-like
object. SockJS gives you a coherent, cross-browser, Javascript API
which creates a low latency, full duplex, cross-domain communication
channel between the browser and the web server, with WebSockets or without.
This necessitates the use of a server, which this is one version of, for Node.js.


# SockJS-node server

SockJS-node is a Node.js server side counterpart of
[SockJS-client browser library](https://github.com/sockjs/sockjs-client).

To install `sockjs-node` run:

    npm install sockjs

A simplified echo SockJS server could look more or less like:

```javascript
const http = require('http');
const sockjs = require('sockjs');

const echo = sockjs.createServer({ prefix:'/echo' });
echo.on('connection', function(conn) {
  conn.on('data', function(message) {
    conn.write(message);
  });
  conn.on('close', function() {});
});

const server = http.createServer();
echo.attach(server);
server.listen(9999, '0.0.0.0');
```

(Take look at
[examples](https://github.com/sockjs/sockjs-node/tree/main/examples/echo)
directory for a complete version.)

Subscribe to
[SockJS mailing list](https://groups.google.com/forum/#!forum/sockjs) for
discussions and support.


# SockJS-node API

The API design is based on common Node APIs like the
[Streams API](https://nodejs.org/api/stream.html) or the
[Http.Server API](https://nodejs.org/api/http.html#http_class_http_server).

## Server class

SockJS module is generating a `Server` class, similar to
[Node.js http.createServer](https://nodejs.org/api/http.html#http_http_createserver_requestlistener)
module.

```javascript
const sockjs_server = sockjs.createServer(options);
```

Where `options` is a hash which can contain:

<dl>
<dt>sockjs_url (string, required)</dt>
<dd>Transports which don't support cross-domain communication natively
   ('eventsource' to name one) use an iframe trick. A simple page is
   served from the SockJS server (using its foreign domain) and is
   placed in an invisible iframe. Code run from this iframe doesn't
   need to worry about cross-domain issues, as it's being run from
   domain local to the SockJS server. This iframe also does need to
   load SockJS javascript client library, and this option lets you specify
   its url (if you're unsure, point it to
   <a href="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js">
   the latest minified SockJS client release</a>, this is the default).
   You must explicitly specify this url on the server side for security
   reasons - we don't want the possibility of running any foreign
   javascript within the SockJS domain (aka cross site scripting attack).
   Also, sockjs javascript library is probably already cached by the
   browser - it makes sense to reuse the sockjs url you're using in
   normally.</dd>

<dt>prefix (string regex)</dt>
<dd>A url prefix for the server. All http requests which paths begins
   with selected prefix will be handled by SockJS. All other requests
   will be passed through, to previously registered handlers.</dd>

<dt>response_limit (integer)</dt>
<dd>Most streaming transports save responses on the client side and
   don't free memory used by delivered messages. Such transports need
   to be garbage-collected once in a while. `response_limit` sets
   a minimum number of bytes that can be send over a single http streaming
   request before it will be closed. After that client needs to open
   new request. Setting this value to one effectively disables
   streaming and will make streaming transports to behave like polling
   transports. The default value is 128K.</dd>

<dt>transports (Array of strings)</dt>
<dd>List of transports to enable. Select from `eventsource`, `htmlfile`,
`jsonp-polling`, `websocket`, `websocket-raw`, `xhr-polling`,
and `xhr-streaming`.</dd>

<dt>jsessionid (boolean or function)</dt>
<dd>Some hosting providers enable sticky sessions only to requests that
  have JSESSIONID cookie set. This setting controls if the server should
  set this cookie to a dummy value. By default setting JSESSIONID cookie
  is disabled. More sophisticated behaviour can be achieved by supplying
  a function.</dd>

<dt>log (function(severity, message))</dt>
<dd>It's quite useful, especially for debugging, to see some messages
  printed by a SockJS-node library. This is done using this `log`
  function, which is by default set to nothing. If this
  behaviour annoys you for some reason, override `log` setting with a
  custom handler.  The following `severities` are used: `debug`
  (miscellaneous logs), `info` (requests logs), `error` (serious
  errors, consider filing an issue).</dd>

<dt>heartbeat_delay (milliseconds)</dt>
<dd>In order to keep proxies and load balancers from closing long
  running http requests we need to pretend that the connection is
  active and send a heartbeat packet once in a while. This setting
  controls how often this is done. By default a heartbeat packet is
  sent every 25 seconds.  </dd>

<dt>disconnect_delay (milliseconds)</dt>
<dd>The server sends a `close` event when a client receiving
  connection have not been seen for a while. This delay is configured
  by this setting. By default the `close` event will be emitted when a
  receiving connection wasn't seen for 5 seconds.  </dd>

<dt>disable_cors (boolean)</dt>
<dd>Enabling this option will prevent
  <a href="https://en.wikipedia.org/wiki/Cross-origin_resource_sharing">CORS</a>
  headers from being included in the HTTP response. Can be used when the
  sockjs client is known to be connecting from the same origin as the
  sockjs server. This also disables the iframe HTML endpoint.</dd>
</dl>


## Server instance

Once you have create `Server` instance you can hook it to the
[http.Server instance](https://nodejs.org/api/http.html#http_class_http_server).

```javascript
var http_server = http.createServer();
sockjs_server.attach(http_server);
http_server.listen(...);
```

`Server` instance is an
[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter),
and emits following event:

<dl>
<dt>Event: connection (connection)</dt>
<dd>A new connection has been successfully opened.</dd>
</dl>

All http requests that don't go under the path selected by `prefix`
will remain unanswered and will be passed to previously registered
handlers. You must install your custom http handlers before calling
`attach`. You can remove the SockJS handler later with `detach`.

## Connection instance

A `Connection` instance supports
[Node Stream API](https://nodejs.org/api/stream.html) and
has following methods and properties:

<dl>
<dt>Property: remoteAddress (string)</dt>
<dd>Last known IP address of the client.</dd>

<dt>Property: remotePort (number)</dt>
<dd>Last known port number of the client.</dd>

<dt>Property: address (object)</dt>
<dd>Hash with 'address' and 'port' fields.</dd>

<dt>Property: headers (object)</dt>
<dd>Hash containing various headers copied from last receiving request
    on that connection. Exposed headers include: `origin`, `referer`
    and `x-forwarded-for` (and friends). We explicitly do not grant
    access to `cookie` header, as using it may easily lead to security
    issues (for details read the section "Authorisation").</dd>

<dt>Property: url (string)</dt>
<dd><a href="https://nodejs.org/api/http.html#http_message_url">Url</a>
    property copied from last request.</dd>

<dt>Property: pathname (string)</dt>
<dd>`pathname` from parsed url, for convenience.</dd>

<dt>Property: prefix (string)</dt>
<dd>Prefix of the url on which the request was handled.</dd>

<dt>Property: protocol (string)</dt>
<dd>Protocol used by the connection. Keep in mind that some protocols
   are indistinguishable - for example "xhr-polling" and "xdr-polling".</dd>

<dt>Property: readyState (integer)</dt>
<dd>Current state of the connection:
   0-connecting, 1-open, 2-closing, 3-closed.</dd>

<dt>write(message)</dt>
<dd>Sends a message over opened connection. A message must be a
  non-empty string. It's illegal to send a message after the connection was
  closed (either after 'close' or 'end' method or 'close' event).</dd>

<dt>close([code], [reason])</dt>
<dd>Asks the remote client to disconnect. 'code' and 'reason'
   parameters are optional and can be used to share the reason of
   disconnection.</dd>

<dt>end()</dt>
<dd>Asks the remote client to disconnect with default 'code' and
   'reason' values.</dd>

</dl>

A `Connection` instance emits the following events:

<dl>
<dt>Event: data (message)</dt>
<dd>A message arrived on the connection. Message is a unicode
  string.</dd>

<dt>Event: close ()</dt>
<dd>Connection was closed. This event is triggered exactly once for
   every connection.</dd>
</dl>

For example:

```javascript
sockjs_server.on('connection', function(conn) {
  console.log('connection' + conn);
  conn.on('close', function() {
    console.log('close ' + conn);
  });
  conn.on('data', function(message) {
    console.log('message ' + conn, message);
  });
});
```

## Footnote

A fully working echo server does need a bit more boilerplate (to
handle requests unanswered by SockJS), see the
[`echo` example](https://github.com/sockjs/sockjs-node/tree/main/examples/echo)
for a complete code.

# Examples

If you want to see samples of running code, take a look at:

 * [./examples/echo](https://github.com/sockjs/sockjs-node/tree/main/examples/echo)
   directory, which contains a full example of a echo server.
 * [./tests/test_server](https://github.com/sockjs/sockjs-node/tree/main/tests/test_server) a standard SockJS test server.


# Connecting to SockJS-node without the client

Although the main point of SockJS it to enable browser-to-server
connectivity, it is possible to connect to SockJS from an external
application. Any SockJS server complying with 0.3 protocol does
support a raw WebSocket url. The raw WebSocket url for the test server
looks like:

 * ws://localhost:8081/echo/websocket

You can connect any WebSocket RFC 6455 compliant WebSocket client to
this url. This can be a command line client, external application,
third party code or even a browser (though I don't know why you would
want to do so).

Note: This endpoint will *not send any heartbeat packets*.


# Deployment and load balancing

There are two issues that need to be considered when planning a
non-trivial SockJS-node deployment: WebSocket-compatible load balancer
and sticky sessions (aka session affinity).

## WebSocket compatible load balancer

Often WebSockets don't play nicely with proxies and load balancers.
Deploying a SockJS server behind Nginx or Apache could be painful.

Fortunately recent versions of an excellent load balancer
[HAProxy](http://haproxy.1wt.eu/) are able to proxy WebSocket
connections. We propose to put HAProxy as a front line load balancer
and use it to split SockJS traffic from normal HTTP data. Take a look
at the sample
[SockJS HAProxy configuration](https://github.com/sockjs/sockjs-node/blob/main/examples/haproxy.cfg).

The config also shows how to use HAproxy balancing to split traffic
between multiple Node.js servers. You can also do balancing using dns
names.

## Sticky sessions

If you plan deploying more than one SockJS server, you must make sure
that all HTTP requests for a single session will hit the same server.
SockJS has two mechanisms that can be useful to achieve that:

 * Urls are prefixed with server and session id numbers, like:
   `/resource/<server_number>/<session_id>/transport`.  This is
   useful for load balancers that support prefix-based affinity
   (HAProxy does).
 * `JSESSIONID` cookie is being set by SockJS-node. Many load
   balancers turn on sticky sessions if that cookie is set. This
   technique is derived from Java applications, where sticky sessions
   are often necessary. HAProxy does support this method, as well as
   some hosting providers, for example CloudFoundry.  In order to
   enable this method on the client side, please supply a
   `cookie:true` option to SockJS constructor.


# Development and testing

If you want to work on SockJS-node source code, you need to clone the
git repo and follow these steps. First you need to install
dependencies:

    cd sockjs-node
    npm install

If compilation succeeds you may want to test if your changes pass all
the tests. Currently, there are two separate test suites.

## SockJS-protocol Python tests

To run it run something like:

    ./scripts/test.sh

For details see
[SockJS-protocol README](https://github.com/sockjs/sockjs-protocol#readme).

## SockJS-client Karma tests

To run it run something like:

    cd sockjs-client
    npm run test:browser_local

For details see
[SockJS-client README](https://github.com/sockjs/sockjs-client#readme).

# Various issues and design considerations

## Authorisation

SockJS-node does not expose cookies to the application. This is done
deliberately as using cookie-based authorisation with SockJS simply
doesn't make sense and will lead to security issues.

Cookies are a contract between a browser and an http server, and are
identified by a domain name. If a browser has a cookie set for
particular domain, it will pass it as a part of all http requests to
the host. But to get various transports working, SockJS uses a middleman
- an iframe hosted from target SockJS domain. That means the server
will receive requests from the iframe, and not from the real
domain. The domain of an iframe is the same as the SockJS domain. The
problem is that any website can embed the iframe and communicate with
it - and request establishing SockJS connection. Using cookies for
authorisation in this scenario will result in granting full access to
SockJS communication with your website from any website. This is a
classic CSRF attack.

Basically - cookies are not suited for SockJS model. If you want to
authorise a session - provide a unique token on a page, send it as a
first thing over SockJS connection and validate it on the server
side. In essence, this is how cookies work.


## Deploying SockJS on Heroku

Long polling is known to cause problems on Heroku, but
[workaround for SockJS is available](https://github.com/sockjs/sockjs-node/issues/57#issuecomment-5242187).
