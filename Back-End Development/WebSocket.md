# WebSocket





### [Differences between TCP sockets and web sockets, one more time](https://stackoverflow.com/questions/16945345/differences-between-tcp-sockets-and-web-sockets-one-more-time)

When you send bytes from a buffer with a normal TCP socket, the send function returns the number of bytes of the buffer that were sent. If it is a non-blocking socket or a non-blocking send then the number of bytes sent may be less than the size of the buffer. If it is a blocking socket or blocking send, then the number returned will match the size of the buffer but the call may block. With WebSockets, the data that is passed to the send method is always either sent as a whole "message" or not at all. Also, browser WebSocket implementations do not block on the send call.

But there are more important differences on the receiving side of things. When the receiver does a recv \(or read\) on a TCP socket, there is no guarantee that the number of bytes returned correspond to a single send \(or write\) on the sender side. It might be the same, it may be less \(or zero\) and it might even be more \(in which case bytes from multiple send/writes are received\). With WebSockets, the receipt of a message is event driven \(you generally register a message handler routine\), and the data in the event is always the entire message that the other side sent.

Note that you can do message based communication using TCP sockets, but you need some extra layer/encapsulation that is adding framing/message boundary data to the messages so that the original messages can be re-assembled from the pieces. In fact, WebSockets is built on normal TCP sockets and uses frame headers that contains the size of each frame and indicate which frames are part of a message. The WebSocket API re-assembles the TCP chunks of data into frames which are assembled into messages before invoking the message event handler once per message.  




