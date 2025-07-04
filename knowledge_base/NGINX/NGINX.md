#nginx #web-server #reverse-proxy #load_balancer 

NGINX is an open source web server written in C and can also be used as a reverse proxy and load balancer

### Web Server
- Serves web content (static/dynamic)
- Listens on an http endpoint and understand how to talk in HTTP

### Reverse Proxy
- Can make NGINX face the internet
- in the back end it will take request from the internet and then move them across our backend appropriately
- Everyone would talk to NGINX and NGINX would turn back (reverse) and talk to other backends
- #### Load balancing
	- Everyone in internet would hit NGINX and it would balance the load across multiple backends
- #### Backend Routing (API Gateway)
	- anything `/payments` go to these set of servers
	- It can morph(change) the request, authenticate the request
	- Does so much stuff and routes it to appropriate backend
	- Checks if a particular backend is down and figures if it has to talk to any other backend
- #### Caching
	- Cache repetitive requests
- #### Rate Limiting


- Any reverse proxy has to be as efficient as possible in the rules it applies in the inspection it does to the packets
- It should be performant and not obstacle


![[NGINX arch.png]]
