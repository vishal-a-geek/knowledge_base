#nginx #Layer-4 #OSI-model #transport-layer #Layer-7 #application-layer #tcp-ip #tcp #http

- OSI Layers:
	- L1 : Physical Layer
	- L2 : Data Link Layer
	- L3: Network Layer
	- L4: Transport Layer (TCP/IP Stack)
	- L5: Session Layer
	- L6: Presentation Layer
	- L7: Application Layer
- In L4 we only see TCP/IP stack, nothing about the app (HTTP)
	- We have access to (Source IP, Source Port) (Destination IP, Destination Port)
		- NGINX / Any Reverse Proxy could restrict based on destination port
	- Sometimes proxies do simple packet inspection
		- Mainly to detect SYN or TLS requests
	- At this layer, it doesn't/can't really look at the content and try to derive GET or POST request
- In L7 we see applications like HTTP/gRPC/WS/SMTP/FTP
	- We have access to more context
	- We can know where the client wants to go, not just IP/Port, but also path, method, headers, body, cookies ...
	- L7 Proxying requires **decryption**
- NGINX can operate in L7 (like HTTP) or Layer 4 (like TCP)

#### Why L4 Proxying?
- L4 Proxying is useful when NGINX doesn't really understand the underlying application layer protocol like Postgres or MySQL
	- NGINX doesn't understand how to terminate such application layer protocols
		- Read/Parse such requests and further turn around to talk to a backend that understands it
		- NGINX can't do that unless it has the capability to understand the application protocol being used
	- If we find ourselves in a situation where NGINX doesn't understand application layer protocols we use such as gRPC, WebRTC, we can opt in for a L4 reverse proxy and can say blindly "Hey blindly just take whatever client gives in the packet/segment, and just forward it to backend"
- Use **stream** context in L4 proxy

#### Why L7 Proxying?
- L7 Proxying is useful when NGINX wants to 
	- share backend connections 
		- Load balancing or sending the request to **not necessarily** one backend server all the time
	- cache results
	- understand and re-write headers (authentication)
	- do more to the content of the request itself
- Load balancers become more efficient at L7
	- but there is a **cost of decryption** and the **cost sharing certificate**
- Use **http** context in L7 proxy

#### HTTP with L4 Proxying
- Can be done. We can say to NGINX that "Hey we know you understand HTTP but please don't look at my content, just merely forward it to backend"
- Such connections cannot be shared
	- i.e. it can only forward to one backend server all the time for a request from particular connection  (sticky connection)
	- since we don't understand HTTP at L4, sharing connection at L4 Proxy can be dangerous (to maintain state/session) and we thus stick to the first backend we use for a request from a particular connection
	- With L7 proxying, since NGINX understands HTTP, which is **stateless**, we can route to any backend what so ever
