## Overview

Sockets are a **method of Interprocess Communications (IPC)** that allows data to be exchanged between applications.

In a client-server scenario, applications communicate using socket like this:

* Each application creates a socket.
* The server binds its socket to a well-known address (name) so that client can locate it.

There are several system calls related:
* socket()
* bind()
* listen()
* accept()
* connect()


Socket I/O can be performed using :
* the conventional **`read()`** and **`write()`** system calls, or 
* using a range of socket-specific system calls: **`send()`**, **`recv`**, **`sendto()`** and **`recvfrom()`**

![alt text](overview_socket_passive_active.png)

## 1. Creating a Socket: `socket()`

```c
#inlcude <sys/socket.h>

int socket(int domain, int type, int protocol);
// It returns file descriptor on success, or -1 on error.
```

There are 3 parameters : **domain**, **type** and **protocol**

### 1.1 Communication domains

* **What is domain?** sockets exist in a communication domain which determines:
	* the method of identifying a socket (i.e. the format of a socket address); and 
	* the range of communication (i.e. either between applications on the same host or on different hosts connected via network)

* **Supported domains**:
	* `AF_UNIX` : allows communication between apps on the **same host**.
	* `AF_INET` : allows communication between apps which are connected via **IPv4 network**.
	* `AF_INET6`: allows communication between apps which are connected via **IPv6 network**.

### 1.2 communication types

2 types:  **Stream** (`SOCK_STREAM`)   and  **Datagram** (`SOCK_DGRAM`)

```
                                     Steam         datagram
                                     
Reliable delivery?                    YES             NO

Message boundaries preserved?          NO             YES

Connection-oriented?                  YES             NO

```

**Stream socket** provides a **reliable**, **bidirectional** **byte-stream** communication channel.

* **Reliable**: means that we are guaranteed that either the transmitted data will arrive intact at the receiving applicaion, or that we'll receive notification of a probable failure in transmission.

* **bidirectional**: means that data can be transmitted in either direction between 2 sockets.

* **connection oriented**: stream sockets operate in connected pairs. 


**Datagram socket** allows data to be exchanged in the form of **message** called **datagrams**. But message may 

* arrive **out of order** or 
* **be duplicated**  or  
* **not arrive at all**.

A datagram socket does not need to be connected (but could be) to another socket in order to be used.


## 2. Binding a socket to an Address: `bind()`

```c
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// return 0 on success, or -1 on error.
```

* **`sockfd`**: a file descriptor obtianed from call to `socket()`.
* **`addr`**: a pointer to a structure, which specifies the address wo which this socket is to be bound.
* **`addrlen`**: an int type to define 


## 3. Listening for incoming connections: `listen()`

```c
#include <sys/socket.h>

int listen(int sockfd, int backlog);
// return 0 on success, or -1 on error.
```
By calling **`listen()`**, it marks the steam socket (defined by file descriptor **`sockfd`**) as **passive**, thus, the socket will subsequently be used to accept connections from other (**active**) sockets.

To understand **`backlog`**, see pic below.

It is possible that a client calls **`connect()`** before the server calls **`accept()`**. This chould happen maybe because the server is busy handling some other client, and this will result in a **`pending connection`**.

So the kernal should keep a record of those information about each pending connection request so that a subsequent **`accept()`** can be processed. The **`backlog`** arguments allows us **to limit the number of such pending connections**.

![alt text](pending_socket_connection.png)

## 4. Accepting a connection: `accept()`

```c
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// return file descriptor on success, or -1 on error
```

* The `accept()` system call accepts an incoming connection o nthe listening stream socket referredto by the file descriptor `sockfd`.
* If there is no pending connections when `accept()` is called, this call to `accept()` **will block** until a connection request arrives.

The key to understand `accept` is that: 
* **it creates a new socket**.
* This **new socket** is connected to the peer socket which performed the `connect()`.
* The **listening socket** (which is defined by `sockfd`) is still open and can be used to accept further connections.


## 5. Connecting to a peer socket: `connect()`

```c
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// return 0 on success, or -1 on error.
```


## all together

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <time.h>
#include <stdbool.h>

int main(int argc, const char * argv[]) {
	// insert code here...
	int listenfd = 0, connectfd = 0;
	struct sockaddr_in serv_addr;

	char sendBuff[] = "Hello there.";

	listenfd = socket(AF_INET, SOCK_STREAM, 0);
	memset(&serv_addr, '0', sizeof(serv_addr));

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_addr.sin_port = htons(2017);

	bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));

	listen(listenfd, 10);

	while(true) {
		connectfd = accept(listenfd, (struct sockaddr*)NULL, NULL);

		write(connectfd, sendBuff, strlen(sendBuff));
		printf("%s\n", sendBuff);
		close(connectfd);
		sleep(1);
	}

	return 0;
}

```

or python version:

```python
import socket


HOST, PORT = '', 2017
MAX_QUEUED_CONNECTIONS = 10
RESPONSE = b"""\
HTTP/1.1 200 OK

Hello there.
"""

listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
listen_socket.bind((HOST, PORT))
listen_socket.listen(MAX_QUEUED_CONNECTIONS)

print('Serving HTTP on port : {}'.format(PORT))

while True:
	print("Before...")
	client_connection, client_address = listen_socket.accept()
	print("After...")
	request = client_connection.recv(1024)
	print("Request: {}".format(request))

	client_connection.sendall(RESPONSE)
	client_connection.close()
```























