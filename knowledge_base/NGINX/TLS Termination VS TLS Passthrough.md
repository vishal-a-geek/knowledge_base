#tls #tls-termination #tls-passthrough #transport-layer-security

## TLS
- TLS : Transport Layer Security
- It is the de-facto way to establish end-to-end encryption between two parties
- Uses symmetric encryption (AES) for communication (client/server has the **same key**)
	- because symmetric encryption is very fast
- Uses asymmetric encryption for **exchanging symmetric keys** (Deffie Hellman)
	- because asymmetric encryption (RSA) is very slow
- Used for authenticating server for the client (browser) by supplying certificate signed by certificate authority

### TLS Termination
- NGINX has TLS (e.g. HTTPS) and backend is not (it's simple HTTP) if it's in a private network
- NGINX terminates TLS and decrypts 
	- and then can either send unencrypted to the backends
	- or can optionally re-write and then re-encrypt the content to the backend

### TLS Passthrough
- Done if we don't trust NGINX or person hosting NGINX for us
- Here, TLS doesn't de-crypt and is just a dumb pipe, which just pipes everything through itself all the way back to backend
	- NGINX here proxies or streams the packets directly to the backend 
- NGINX cannot see anything here
- TLS handshake is all the way forwarded to the backend, like a tunnel
- Can no longer cache, here we can only do L4 check but more secure than re-encrypting with TLS Termination, as here NGINX doesn't need backend certificate
- NGINX here cannot share backend connect. It has to make private connection for every request
	- i.e. always use same/first backend server for all requests of a particular connection