go-socket.io
============

The `socketio` package is a simple abstraction layer for different web browser-
supported transport mechanisms. It is meant to be fully compatible with the
[Socket.IO client](http://github.com/LearnBoost/Socket.IO) JavaScript-library by
[LearnBoost Labs](http://socket.io/), but through custom formatters it should
suit for any client implementation.

It provides an easy way for developers to rapidly prototype with the most
popular browser transport mechanism today:

- [HTML5 WebSockets](http://dev.w3.org/html5/websockets/)
- [XHR Polling](http://en.wikipedia.org/wiki/Comet_%28programming%29#XMLHttpRequest_long_polling)
- [XHR Multipart Streaming](http://en.wikipedia.org/wiki/Comet_%28programming%29#XMLHttpRequest)

## Disclaimer

**The go-socket.io is still very experimental, and you should consider it as an
early prototype.**

## Crash course

The `socketio` package works hand-in-hand with the standard `http` package (by
plugging itself into a configurable `http.ServeMux`) and hence it doesn't need a
full network port for itself. It has an callback-style event handling API. The
callbacks are:

- *SocketIO.OnConnect*
- *SocketIO.OnDisconnect*
- *SocketIO.OnMessage*

Other utility-methods include:

- *SocketIO.Mux*
- *SocketIO.Broadcast*
- *SocketIO.BroadcastExcept*
- *SocketIO.IterConns*
- *SocketIO.GetConn*

Each new connection will be automatically assigned an unique session id and
using those the clients can reconnect without losing messages: the server
persists clients' pending messages (until some configurable point) if they can't
be immediately delivered. All writes are by design asynchronous and can be made
through *Conn.Send*.

Finally, the actual format on the wire is described by a separate `Formatter`.
The default formatter is compatible with the LearnBoost's
[Socket.IO client](http://github.com/LearnBoost/Socket.IO).

## Example: A simple chat server

	package main

	import (
		"http"
		"log"
		"socketio"
	)

	func main() {
		sio := socketio.NewSocketIO(nil)
		sio.Mux("/socket.io/", nil)

		http.Handle("/", http.FileServer("www/", "/"))

		sio.OnConnect(func(c *socketio.Conn) {
			sio.Broadcast(struct{ announcement string }{"connected: " + c.String()})
		})

		sio.OnDisconnect(func(c *socketio.Conn) {
			sio.BroadcastExcept(c,
				struct{ announcement string }{"disconnected: " + c.String()})
		})

		sio.OnMessage(func(c *socketio.Conn, msg string) {
			sio.BroadcastExcept(c,
				struct{ message []string }{[]string{c.String(), msg}})
		})

		log.Stdout("Server started.")
		if err := http.ListenAndServe(":8080", nil); err != nil {
			log.Stdout("ListenAndServe: %s", err.String())
			os.Exit(1)
		}
	}

## tl;dr

You can get the code and run the bundled example by following these steps:

	$ git clone git://github.com/madari/go-socket.io.git
	$ cd go-socket.io
	$ git submodule update --init
	$ cd example
	$ make
	$ ./example

## License 

(The MIT License)

Copyright (c) 2010 Jukka-Pekka Kekkonen &lt;karatepekka@gmail.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
