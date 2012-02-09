#Documentation for libinetsocket

##Clients
###`create_isocket()`

	int create_isocket(const char* host, const char* service, char proto_osi4, char proto_osi3)
	
`create_isocket()` creates and connects a new socket. The parameters have to be filled like this:

* `host` is a string (0-terminated) containing the hostname or IP address of the target host
* `service` is a string (0-terminated) containing the portname or service (e.g. `http`)
* `proto_osi4` is a value representing the protocol on OSI layer 4 which we want to use (defined as macros): `TCP`, `UDP` or `BOTH` when libsocket should choose
* `proto_osi3` is a value representing the protocol on OSI layer 3 which we want to use (defined as macros): `IPv6`, `IPv4` or `BOTH` when libsocket should choose

The return value is either a valid file descriptor on which you can execute `write()` or `read()` (from `unistd.h`). If there's an error when creating
that socket, it returns -1.

Important for UDP sockets: The returned socket file descriptor may be used with `read()` and `write()` as well as TCP sockets. If you want to use `sendto()` and
`recvfrom()`, don't use this library. But generally, it is not necessary to use sendto and recvfrom: When you use connected datagram sockets,
each `read()` call will create a new datagram.

###`reconnect_isocket()`

	int reconnect_isocket(int sfd, char* host, char* service, int socktype);

`reconnect_isocket()` is an easy way to connect an existing socket to a new peer. It is faster and more elegant than destroying the first socket with `destroy_isocket()`
and make a new with `create_isocket()`. The parameters:

* `sfd` is the existing Socket File Descriptor
* `host` and `service` like in `create_isocket()`
* `socktype` is either TCP or UDP (the macro constants from `create_isocket()`). It has to be the type 
which you chose at `create_isocket()` because it is impossible to get the socktype from the file descriptor.

This function returns either 0 (Success) or -1 (Failure).

###`shutdown_isocket()` 

	int shutdown_isocket(int sfd, int method)

`shutdown_isocket()` shuts a socket down. This means that (READ) you cannot read data anymore respectively (WRITE) you cannot write data anymore
and the other peer gets an EOF signal (`read()` returns 0).

* `sfd` is the Socket File Descriptor
* `method` is either READ, WRITE or READ|WRITE (ORed).

This function returns either 0 (Success) or -1 (Failure)

###`destroy_isocket()`
	
	int destroy_isocket(int sfd)

`destroy_isocket()` `close()`s the specified Socket file descriptor `sfd`. It does not perform a `shutdown()` on it because that would produce
an error if you shut it down before. But that's equal because `close()` also sends EOF.

This function returns either 0 (Success) or -1 (Failure)

##Server

libsocket also supports INET server sockets (also called Passive Sockets).

###`create_issocket()`
'issocket' stands for 'Internet Server Socket', not for 'is socket'. :)

	int create_issocket(const char* bind_addr, const char* bind_port, char proto_osi4, char proto_osi3);

Creates a new server socket:

* `bind_addr` is the address to bind to. It's normally good when you specify "0.0.0.0".
* `bind_port` is the port to bind to.
* proto_osi4 is either TCP or UDP. If it's UDP, the socket doesn't `listen()`.
* proto_osi3 is either IPv4 or IPv6

The call to `listen()` (which is not executed when using UDP sockets uses the biggest linux backlog size 128. You should change it in the source code if you
want another value.

This function returns an int value suitable for calls with `accept()` and `accept_issocket()`.

###`accept_issocket()`

 	int accept_issocket(int sfd, char* src_host, size_t src_host_len, char* src_service, size_t src_service_len, int flags);

`accept_issocket()` performs an action similar to `accept()`. **It may not be called on UDP server sockets.** 

* `sfd` is the socket file descriptor
* `src_host` is a pointer to a buffer into which the lib writes the hostname of the client. `src_host_len` is the length of its buffer. More bytes are truncated
* `src_service` and `src_service_len` is the same like `src_host` and `src_host_len`, but for the ports
* `flags` may be `NUMERIC`, which results in numeric host and service names.

`accept_issocket()` returns a file descriptor for a connection to the client which is to be used with `read()` and `write()`. 
In case of failure, it returns -1. `accept_issocket()` blocks until a client connects.


#Documentation for libunixsocket

##Client

###`create_socket()`

	int create_usocket(const char* path, int socktype);

Creates and connects a new UNIX domain socket file descriptor for a socket located at `path`, type `socktype`.
`socktype` is either `STREAM` or `DGRAM`. 

Important for DGRAM sockets: Please think twice if you want to use DGRAM sockets in UNIX domain. They do not have any advantages
over STREAM sockets!

###`shutdown_socket()` 

	int shutdown_socket(int sfd, int method)

`shutdown_socket()` shuts a socket down. This means that (READ) you cannot read data anymore respectively (WRITE) you cannot write data anymore
and the other peer gets an EOF signal (`read()` returns 0).

* `sfd` is the Socket File Descriptor
* `method` is either READ, WRITE or READ|WRITE (ORed).

###`destroy_socket()`
	
	int destroy_socket(int sfd)

`destroy_socket()` shuts the socket down for READ and WRITE operations and `close()`s it.


#Compile options

If you specify the flag `-DVERBOSE` at compile time, libsocket uses STDERR to print information about occurred errors.
VERBOSE is not activated by default so libsocket is 'quiet'.