# P2P-application
In this project, a peer-to-peer application was developed in C. This application is focused on file upload and download. There is an index server, and multiple peers can connect to it and share files with each other. To upload a file, the peers create their own server and send the index server information on which IP address and port the file can be downloaded from. To download a file, a peer sends the file name to the index server, which returns an IP address and port from which the content can be downloaded. Hence, the index server is for communicating information on the different files that can be downloaded. Peers can also request the index server to send a list of available files and to deregister files they have previously registered.

The User Datagram Protocol (UDP) is used for messages between the peer and client and Transmission Control Protocol (TCP) is used for file upload/download between peers. TCP is a connection-oriented protocol and UDP is a connectionless protocol. UDP is faster but there is no retransmission on error, hence this is a better protocol for processing the user’s commands because if there are errors the user will know immediately and can easily reenter the commands. TCP is slower but detects errors more easily and guarantees retransmission, which is essential for its purposes because a faulty data packet can ruin an entire file.


# Description of the client and server programs
The client and server communicate with a data unit. The protocol data unit (PDU) has the format of 1 byte for type and a fixed predetermined length for data. In this project, we used a length of 100 bytes for the data. This is a sufficient length for messages between the index server and peers, but for file transfer between two peers the message (file data) needs to be split into multiple PDUs. The format of the PDU is shown in the figure below:
 

All communication between the client and the server uses UDP connections, while communication between clients is done with a TCP connection. 

Peer
The peer uses a linked list to store information about local files that have been registered. The linked list stores the names of the files. The interface allows the user to select the following options:
●	R - to register a file. The user enters the file name and the program starts by creating a TCP server which is responsible for receiving requests from other servers to download the file. Then, it creates a string which contains the file name, peer name and port number in the 40 characters, in the format shown below. The string is sent in a PDU of type ‘R’ to the index server and if the response is ‘A’ (acknowledgement), a child process is created to run the TCP server and receive download requests for the file.
 

●	D - to download a file. The user enters the file name. The program then sends a PDU of type ‘S’ with the peer name (of the requestor) and file name in the data field in the format shown below. If the response is a PDU of type ‘S’, this means that the response’s data field contains an IP address and port number where this file can be downloaded from. Then, the program downloads the file from that IP address and port number and creates its own TCP server to host the file separately.
 

●	T - to deregister a local file. The user enters the file name to deregister and the program sends this in a T-type PDU. 
●	L - to list local contents. This prints the contents of the linked list.
●	O - to print a list of all files registered at the index server. This sends a request of type ‘O’ to the index server, and the index server returns a PDU of type ‘O’ containing the file names.
●	Q - to quit. This iterates over the linked list and deregisters all the local files. 

The function to create the TCP server generates a random available port number. To generate a random port number, it assigns htons(0) to the sin_port field of the sockaddr_in associated with the server.

The function to run the TCP server creates a child process which checks for new requests using the accept() function. If a new request is found it executes the send_file command to send the file.

Functions to add and delete from the linked list were created.

Server
We implemented a Look Up Table (LUT) mechanism in the index server by storing records of the peer’s username, IP address, and port using a structure called Content. The LUT stores the data in a file named Content.txt so that it can be dynamically expanded and scaled down. We made several functions to append, search, delete, update, count, and display content from the file.

The index server’s flow starts by creating a socket, then binding the socket, and then goes in an infinite loop to perform the main logic of the server. The loop starts by a recvfrom() to wait for data from the client. Once the data is received, it saves it in a structure called PDU and compares the type of the message received. The type of the message received then determines how the program acts, and this was implemented using if-else statements. 
The PDU types that are handled by the server are R (Content Registration), T (Content Deregistration), S (Search Content), O (display registered content), and E (Error). The PDU type functionalities are as stated below: 
●	R (Content Registration): The index server checks its LUT for the name of the file, if it doesn't exist then it stores the name of the file along with the port number, IP address, and the username of the sender and sends back an acknowledgment message back to the client with the port number associated with the file. If the file already exists in the LUT, then the index server will send an Error message telling the client that the file already exists.
●	T (Content Deregistration): The index server checks the LUT for the name of the file and the name of the username associated with the file. If the file was registered by the same username of the sender then it removes the file from the LUT and returns an Acknowledgement message back to the sender. Otherwise, it sends back an error message.
●	S (Search Content): The index server searches for the file from the LUT and if found then it sends back an acknowledgment message back to the server with the IP address and port number associated with the content server of the file. Otherwise, it sends back an Error message back to the sender. This is used when a client requests to download a specific file.
●	O (display registered content): The index server sends back a list of the content stored in the LUT at the moment of receiving the request.
●	E (Error): An error message is sent for various reasons explained above, and it serves as an indicator that the requested execution could not be done correctly.


# Background Information on Socket Programming
Socket programming is the interface for network communications between processes. It is a way to connect different nodes with each other for the purpose of sharing information between them. 

The stages of socket programming start with the creation of a socket using the system call socket (domain, type, protocol). The domain determines the communication domain such as IPv4 or IPv6 and in our application we used the IPv4 domain which is called AF_INET. The socket type declares the type of communication such as UDP or TCP. In our application we used both types and declared them using SOCK_STREAM for TCP connections and SOCK_DGRAM for UDP connections. The protocol value determines the protocol used, which in our case it was IP and it is indicated by the number 0. 

The second stage would be to create an address that the socket can connect to using a built in structure called sockaddr_in that needs to be configured with the address, port, and family.

The third stage is binding the address with the socket we created by using the bind(socket, (struct sockaddr *)&socket_address, sizeof(socket_address)) function. 

Once these three stages are complete, we can use the socket for communication using different built in functions like listen(),connect(), accept(),read(), write(), recvfrom(),sendto(). Many of these functions were used in the application to accomplish the logic required for a peer-to-peer communication.
