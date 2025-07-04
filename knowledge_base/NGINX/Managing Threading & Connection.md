#threading #connection #tcp #http #socket

#### Listener Process/Thread:
- **Socket**: (IP-Port) pair on which the backend applications listen
	- Socket is not a connection, but a place to connect
	- Just like a wall socket, where we can plug in connections
- Socket lives in a **process** called **Listener**

#### Acceptor Process/Thread:
- With socket in hand, backend application calls OS function `accept(socket)` with the socket in hand, for accepting any available connection on this socket (ip-port)
- `accept(socket)` returns a file descriptor representing a connection
- connections have to be actively accepted by the application in order to serve clients
- else, connections will remain in the OS accept-queue unused
- Acceptor is the thread/process that calls accept function

#### Reader (Worker) Process/Thread:
- Process that reads the data sent to this connection from the buffer
- else the buffer for this connection allocated by OS will fill and client will not be able to send more data unless buffer has not full (TCP)

#### TCP Stream Vs HTTP Requests
- TCP is a streaming protocol. 
- When we send a GET request from frontend (say Dio), Dio creates a TCP connection (if one doesn't exist) and build out the HTTP request consisting of method, protocol version, headers, URL parameters etc
- HTTP request is well defined with start and end
- TCP stream is just raw bytes of data
- Reader process is responsible to read all the TCP stream and "look" for requests, recognising the HTTP request start and end
- This collection of bytes is now translated (**parses**) to a logical request
- This request is then delivered to the L7 for processing
- This **parsing** takes a toll on whatever thread that does it

