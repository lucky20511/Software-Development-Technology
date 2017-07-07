# WebSocket

### [Differences between TCP sockets and web sockets, one more time](https://stackoverflow.com/questions/16945345/differences-between-tcp-sockets-and-web-sockets-one-more-time)

When you send bytes from a buffer with a normal TCP socket, the send function returns the number of bytes of the buffer that were sent. If it is a non-blocking socket or a non-blocking send then the number of bytes sent may be less than the size of the buffer. If it is a blocking socket or blocking send, then the number returned will match the size of the buffer but the call may block. With WebSockets, the data that is passed to the send method is always either sent as a whole "message" or not at all. Also, browser WebSocket implementations do not block on the send call.

But there are more important differences on the receiving side of things. When the receiver does a recv \(or read\) on a TCP socket, there is no guarantee that the number of bytes returned correspond to a single send \(or write\) on the sender side. It might be the same, it may be less \(or zero\) and it might even be more \(in which case bytes from multiple send/writes are received\). With WebSockets, the receipt of a message is event driven \(you generally register a message handler routine\), and the data in the event is always the entire message that the other side sent.

Note that you can do message based communication using TCP sockets, but you need some extra layer/encapsulation that is adding framing/message boundary data to the messages so that the original messages can be re-assembled from the pieces. In fact, WebSockets is built on normal TCP sockets and uses frame headers that contains the size of each frame and indicate which frames are part of a message. The WebSocket API re-assembles the TCP chunks of data into frames which are assembled into messages before invoking the message event handler once per message.



# Current Web Protocol

Here is a framework for thinking about web protocols:

* **TCP**
  : low-level, bi-directional, full-duplex, and guaranteed order transport layer. No browser support \(except via plugin/Flash\).
* **HTTP 1.0**
  : request-response transport protocol layered on TCP. The client makes one full request, the server gives one full response, and then the connection is closed. The request methods \(GET, POST, HEAD\) have specific transactional meaning for resources on the server.
* **HTTP 1.1**
  : maintains the request-response nature of HTTP 1.0, but allows the connection to stay open for multiple full requests/full responses \(one response per request\). Still has full headers in the request and response but the connection is re-used and not closed. HTTP 1.1 also added some additional request methods \(OPTIONS, PUT, DELETE, TRACE, CONNECT\) which also have specific transactional meanings. However, as noted in the
  [introduction](http://tools.ietf.org/html/draft-ietf-httpbis-http2-01#section-1)
  to the HTTP 2.0 draft proposal, HTTP 1.1 pipelining is not widely deployed so this greatly limits the utility of HTTP 1.1 to solve latency between browsers and servers.
* **Long-poll**
  : sort of a "hack" to HTTP \(either 1.0 or 1.1\) where the server does not response immediately \(or only responds partially with headers\) to the client request. After a server response, the client immediately sends a new request \(using the same connection if over HTTP 1.1\).
* **HTTP streaming**
  : a variety of techniques \(multipart/chunked response\) that allow the server to send more than one response to a single client request. The W3C is standardizing this as
  [Server-Sent Events](http://en.wikipedia.org/wiki/Server-sent_events)
  using a
  `text/event-stream`
  MIME type. The browser API \(which is fairly similar to the WebSocket API\) is called the EventSource API.
* **Comet/server push**
  : this is an umbrella term that includes both long-poll and HTTP streaming. Comet libraries usually support multiple techniques to try and maximize cross-browser and cross-server support.
* **WebSockets**
  : a transport layer built-on TCP that uses an HTTP friendly Upgrade handshake. Unlike TCP, which is a streaming transport, WebSockets is a message based transport: messages are delimited on the wire and are re-assembled in-full before delivery to the application. WebSocket connections are bi-directional, full-duplex and long-lived. After the initial handshake request/response, there is no transactional semantics and there is very little per message overhead. The client and server may send messages at any time and must handle message receipt asynchronously.
* **SPDY**
  : a Google initiated proposal to extend HTTP using a more efficient wire protocol but maintaining all HTTP semantics \(request/response, cookies, encoding\). SPDY introduces a new framing format \(with length-prefixed frames\) and specifies a way to layering HTTP request/response pairs onto the new framing layer. Headers can be compressed and new headers can be sent after the connection has been established. There are real world implementations of SPDY in browsers and servers.
* **HTTP 2.0**
  : has similar goals to SPDY: reduce HTTP latency and overhead while preserving HTTP semantics. The current draft is derived from SPDY and defines an upgrade handshake and data framing that is very similar the the WebSocket standard for handshake and framing. An alternate HTTP 2.0 draft proposal \(httpbis-speed-mobility\) actually uses WebSockets for the transport layer and adds the SPDY multiplexing and HTTP mapping as an WebSocket extension \(WebSocket extensions are negotiated during the handshake\).
* **WebRTC/CU-WebRTC**
  : proposals to allow peer-to-peer connectivity between browsers. This may enable lower average and maximum latency communication because as the underlying transport is SDP/datagram rather than TCP. This allows out-of-order delivery of packets/messages which avoids the TCP issue of latency spikes caused by dropped packets which delay delivery of all subsequent packets \(to guarantee in-order delivery\).
* **QUIC**
  : is an experimental protocol aimed at reducing web latency over that of TCP. On the surface, QUIC is very similar to TCP+TLS+SPDY implemented on UDP. QUIC provides multiplexing and flow control equivalent to HTTP/2, security equivalent to TLS, and connection semantics, reliability, and congestion control equivalentto TCP. Because TCP is implemented in operating system kernels, and middlebox firmware, making significant changes to TCP is next to impossible. However, since QUIC is built on top of UDP, it suffers from no such limitations. QUIC is designed and optimised for HTTP/2 semantics.

### A History of Polling {#a-history-of-polling}

Realtime web applications have been with us for quite some time now. To the end user, these applications feel responsive and fluid. Gmail is \(arguably\) one of the most major applications to popularize this technique. JavaScript is at the heart here.

Have you ever been in the middle of replying to an email, when \(suddenly\) you’re notified the person has sent you another followup? That’s the perfect example of polling - sometimes referred to as server-push or comet technology.

### A Tale of Two Polling Techniques {#a-tale-of-two-polling-techniques}

#### Traditional Polling {#traditional-polling}

_The setInterval Technique_

Now, if you needed to poll your web service, your first instinct might be to do something like this with JavaScript and jQuery:

```
setInterval(function(){
    $.ajax({ url: "server", success: function(data){
        //Update your dashboard gauge
        salesGauge.setValue(data.value);
    }, dataType: "json"});
}, 30000);
```

Here, we have the poll ready to execute every thirty \(30\) seconds. This code is pretty good. It’s clean and asynchronous. You’re feeling confident. Things will work \(and they will\), but with a catch. What if it takes longer than thirty \(30\) seconds for the server to return your call?

That’s the gamble with using_setInterval_. Lag, an unresponsive server or a whole host of network issues could prevent the call from returning in its allotted time. At this point, you could end up with a bunch of queued Ajax requests that won’t necessarily return in the same order.

_The setTimeout Technique_

If you find yourself in a situation where you’re going to bust your interval time, then a recursive_setTimeout_pattern is recommend:

```
(function poll(){
   setTimeout(function(){
      $.ajax({ url: "server", success: function(data){
        //Update your dashboard gauge
        salesGauge.setValue(data.value);

        //Setup the next poll recursively
        poll();
      }, dataType: "json"});
  }, 30000);
})();
```

Using the Closure technique,_poll_becomes a self executing JavaScript function that runs the first time automatically. Sets up the thirty \(30\) second interval. Makes the asynchronous Ajax call to your server. Then, finally, sets up the next poll recursively.

As you can see, jQuery’s Ajax call can take as long as it wants to. So, this pattern doesn’t guarantee execution on a fixed interval per se. But, it does guarantee that the previous interval has completed before the next interval is called.

Both techniques suffer from the same flaw - a new connection to the server must be opened each time the_$.ajax_method is called. To make that connection, your realtime app must gear up and battle through hoards of competing network traffic to make it to your server.

What if you could just keep the connection open?

#### Long Polling - An Efficient Server-Push Technique {#long-polling---an-efficient-server-push-technique}

**EDIT:**Applications built with Long Polling in mind attempt to offer realtime server interaction, using a persistent or[long-lasting HTTP connection](http://en.wikipedia.org/wiki/Comet_%28programming%29)between the server and the client.

Sadly, with our current technology, long polling cannot be accomplished with client-side JavaScript alone. What you really need is a framework that combines HTML5 WebSockets with some server-side component.

If you’re trying to do long polling with JavaScript only - STOP. It’s not possible without a server-side piece like SignalR for .NET or Socket.IO for Node.JS applications.

If you still want to do some type of client-side polling, then better to use setTimeout to set the interval in some fashion. Maybe something like:

```
(function poll() {
   setTimeout(function() {
       $.ajax({ url: "server", success: function(data) {
            sales.setValue(data.value);
       }, dataType: "json", complete: poll });
    }, 30000);
})();
```

If you can pull off keeping your connection open, then your application could see faster server response times and feel more responsive. That’s a good thing - Good luck!

Thanks to @Lars for pointing this out in the comments. Thanks man!

### What’s next? HTML5 WebSockets. {#whats-next-html5-websockets}

These types of Ajax Push techniques set the foundation for HTML5 WebSockets. With HTML5 WebSockets, we’ll be able to see true Server Push styles of application development. This will make for truly responsive web applications.

Lately, I’ve been recommending[Socket.IO](http://socket.io/)for just such the occasion. Think of Socket.IO as the jQuery of HTML5 WebSockets.

No two browser vendors will implement the WebSockets protocol exactly alike. Socket.IO tries to normalize those differences. Here’s how you might use Socket.IO with our[Dashboard Gauge Suite](http://techoctave.com/gauges):

```
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io.connect("http://localhost");
  socket.on('sales', function (data) {
    //Update your dashboard gauge
    salesGauge.setValue(data.value);

    socket.emit('profit', { my: 'data' });
  });
</script>
```

In order to provide realtime connectivity on every browser, Socket.IO selects the most capable transport at runtime, without it affecting the API. If WebSockets are available, it will use WebSockets.

If WebSockets aren’t available, Socket.IO will select the next best transport including: Adobe Flash Socket or Ajax Polling. So having a solid understanding of JavaScript Long Polling examples is crucial.

We’re still some time off before WebSockets will be universally and consistently supported. Until then, the jQuery Long Polling technique is a best-in-class solution for realtime server communications.

### Eat, Pray and Code {#eat-pray-and-code}

Long polling addresses the weakness of traditional polling by keeping the connection to your server open. Keeping the connection to the server open eliminates the travel time from client to server and thus, significantly reduces the issues surrounding network latency.

If you’re looking for a beautiful suite of dashboard gauges for your next business intelligence project, I encourage you to check out our[Dashboard Gauge Suite](http://techoctave.com/gauges). And if you need an accompanying Polling solution, please do give the jQuery Long Polling solution a try.

We also sell a beautiful set of JavaScript Charts in our[JavaScript Charts](http://techoctave.com/charts)Suite. It’s easy to get started, with a powerful API when you need it.

  


