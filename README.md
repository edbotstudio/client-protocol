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

Messages are sent and received over the connection in JSON format. Each message has an integer **category**
property defined here.

|Category|Name|Description|
|---|:---|:---|
|1|REQUEST|Request sent from client to server
|2|RESPONSE|Request response from the server
|3|UPDATE|Sent from server to client when data is updated (see below)
|4|DELETE|Sent from server to client when data is deleted (see below)
|5|CLOSE|Sent from server to client when the server closes the connection

### Request & Response Messages

Example request message, **params** content omitted:

```json
{
    "category": 1,
    "sequence": 1,
    "type": 1,
    "params": {
    }
}
```

Example response message, **data** content omitted:

```json
{
    "category": 2,
    "sequence": 1,
    "type": 1,
    "status": {
        "success": true,
        "text":"OK"
    },
    "data": {
    }
}
```

As you can see, request and response messages additionally have an integer **sequence** and **type** property.
The sequence number is necessary because the messaging protocol is asynchronous, meaning a particular response
does not always directly follow a matching request. Some requests take longer than others to complete.

The client should generate a sequence number, unique to this connection, for each request message. It's a good
idea to start with a sequence number of 1 and then increment for subsequent requests. The matching response
will contain the same sequence (and type) number.

The message types are listed below. The meaning of some types are robot specific and described in the next section.

|Type|Name|Description
|---|:---|:---|
|1|INIT|Initialise the client.
|2|GET_CLIENTS|Get clients connected to this server|
|3|GET_SERVERS|Get the available servers|
|4|GET_SENSORS|Get the sensor values|
|10|RUN_MOTION|Run a motion|
|11|SET_SERVOS|Set servo parameters|
|20|SET_SPEAKER|Set the speaker parameters|
|21|SET_DISPLAY|Set the display parameters|
|22|SET_OPTIONS|Set connection specific options|
|23|SET_CUSTOM|Set a custom value|
|24|SAY|Speak the text|
|25|RESET|Reset to initial values|

## Licence

Distributed under the MIT Licence. See the [LICENCE](../main/LICENCE) for more information.
