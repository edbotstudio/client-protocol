# EdbotStudio Client Protocol

Documentation for the EdbotStudio Client Protocol.

## Introduction

This document defines the communication protocol between a client program and the Edbot Studio server.
The protocol is used to query and control the robots configured on the server. The document is intended
for developers wishing to add support for new programming languages or to offer an alternative
implementation of an existing language. It should be read in conjunction with existing client code to
gain a full understanding of the protocol. The client code can be found in the following repos:

https://github.com/edbotstudio/client-python  
https://github.com/edbotstudio/client-nodejs

## Connecting

The client program (client) should connect to the Edbot Studio server (server) using as Web socket. Most
modern languages have built-in support for Web sockets which provide fast full-duplex communication
between client and server, maintained until the socket is closed. This is commonly referred to as
_push technology_. Use the following URL to connect to to the server:

ws://_server_:_port_/api

Commonly _server_ is **localhost**, however it could be the name or IP address of a remote server.
By default the _port_ number is **54255** but it may be different depending on the server configuration.

## Messaging

Messages are sent and received over the connection in JSON format. Each message has an integer **sort**
property defined here.

|Sort|Name|Description|
|---|:---|:---|
|1|SORT_REQUEST|Request sent from client to server
|2|SORT_RESPONSE|Request response from the server
|3|SORT_UPDATE|Sent from server to client when data is updated (see below)
|4|SORT_DELETE|Sent from server to client when data is deleted (see below)
|5|SORT_CLOSE|Sent from server to client when server closes connection

## Licence

Distributed under the MIT Licence. See the [LICENCE](../main/LICENCE) for more information.
