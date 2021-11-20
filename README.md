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

## Messages

Messages are sent and received over the connection in JSON format. Each message has an integer **category**
property defined here.

|Category|Name|Description|
|---|:---|:---|
|1|REQUEST|Request sent from client to server
|2|RESPONSE|Request response from the server
|3|UPDATE|Sent from server to client when data is added or updated (see below)
|4|DELETE|Sent from server to client when data is deleted (see below)

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
        "text": "OK"
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

The response message also contains a **status** object comprising a **success** property indicating whether the
request was successful or not. If unsuccessful, the **text** property holds the reason for the failure. This may
be extended in future by adding a integer **code** property to enumerate the error.

The message types are listed below. The meaning of some types are robot specific.

| Type | Name | Description |
| :--- | :--- | :--- |
| 1 | INIT | Initialise the client |
| 2 | GET_CLIENTS | Get clients connected to this server |
| 3 | GET_SERVERS | Get the available servers |
| 4 | GET_SENSORS | Get the sensor values |
| 5 | RUN_MOTION | Run a motion |
| 6 | SET_SERVOS | Set servo parameters |
| 7 | SET_SPEAKER | Set the speaker parameters |
| 8 | SET_DISPLAY | Set the display parameters |
| 9 | SET_OPTIONS | Set connection specific options |
| 10 | SET_CUSTOM | Set a custom value |
| 11 | SAY | Speak the text |
| 12 | RESET | Reset to initial values |

---

**INIT**

Initialise the client. The INIT request should be made directly after connecting and before
any subsequent requests. The message params are described below.

| Param | Type | Optional | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| name | string | yes | null | A name for this client connection |
| reporters | boolean | yes | true | Send reporter updates to this client |
| deviceAlias | string | yes | null | An alias for this client device |

Example INIT request:

```yaml
{
    "category": 1,
    "sequence": 1,
    "type": 1,
    "params": {
        "name": "My Session",
        "reporters": true,
        "deviceAlias": "My Desktop PC"
    }
}
```

The INIT response contains server information in the **data** property described below. The client can store
this data and keep it up-to-date by processing [update messages](#update--delete-messages) from the server.

The data hierarchy is constructed from JSON objects and avoids JSON arrays. This makes it easier and quicker
for the client to find a particular property for update or delete.

Example INIT response with annotation:

```yaml
{
    "category": 2,
    "type": 1,
    "sequence": 1,
    "status": {
        "success": true,
        "text": "OK"
    },
    "data": {
        "server": {
            "remote": false,                                    // is server remote to this client?
            "version": "5.2.0.1597",                            // server software version
            "platform": "Windows 10, 10.0.19042.928, amd64",    // server hardware platform
            "endpoint": "paprika.lan:54255"                     // server endpoint
        },
        "session": {                                            // client session info
            "id": "aOhvF8Pu",                                   // internal id for this session
            "name": "My Session",                               // optional name for this session
            "device": {                                         // client device
                "id": "paprika.lan",                            // device id
                "name": "My Desktop PC (paprika.lan)",          // device name with alias if provided
                "remote": false                                 // is server remote to this device?
            }
        },
        "devices": {                                            // devices connected since startup
            "paprika.lan": {                                    // device id as key
                "id": "paprika.lan",
                "name": "My Desktop PC (paprika.lan)",
                "remote": false
            },
            "pepper.lan": {
                "id": "pepper.lan",
                "name": "My Laptop PC (pepper.lan)",
                "remote": true
            }
        },
        "robots": {                                             // robots configured on server
            "Anna": {                                           // robot name
                "control": "paprika.lan",                       // controlling device
                "model": {                                      // robot model
                    "name": "Edbot Mini",
                    "type": "edbot-mini",
                    "category": "Humanoid",                     // currently unused category
                    "thumb": "edbot-mini.gif",
                    "connectors": [ "btc" ],                    // array of supported connector types
                    "defaultMotionFile": "C:\\motions.mtnx"     // default motion file or null
                },
                "connector": {                                  // connector used for this robot
                    "type": "btc",                              // connector type
                    "name": "Bluetooth"                         // connector name
                    "start": false,                             // connect on software startup?
                    "status": 1,                                // disconnected, connecting, connected
                    "device": {                                 // device specific to connector type
                        "id": "00:16:53:7f:29:1b",
                        "name": "ROBOTIS BT-210"
                    }
                },
                "speech": {                                     // robot speech configuration
                    "voice": {                                  // speech voice
                        "id": "0",
                        "name": "Microsoft Zira"
                    },
                    "device": {                                 // speech device
                        "id": "0",
                        "name": "Sound blaster"
                    },
                    "volume": 100,                              // volume 0 -> 100
                    "rate": 0,                                  // rate -10 -> 10
                    "pitch": 0                                  // pitch -10 -> 10
                },
                "motions": {                                    // motions, null if not supported
                    "motionFile": "C:\\motions.mtnx",           // actual motion file for this robot
                    "groups": {                                 // motion groups
                        "All": [                                // array of motion { id, name } objects
                            { "id": 6, "name": "Bow 2"},
                            // ...
                        ],
                        "Basic": [
                            { "id": 6, "name": "Bow 2"},
                        ],
                        // ...
                    }
                }
                "reporters": {                                  // reporters - may change frequently
                    "servos": {                                 // servo reporters, can be null
                        "1": 512,
                        "2": 384
                    },
                    "sensors": {                                // sensor reporters, can be null
                        "1": 34
                    }
                }
            }
        }
    }
}
```

---

**GET_CLIENTS**

Get the client sessions connected to this server.

Example GET_CLIENTS request:

```yaml
{
    "category": 1,
    "sequence": 2,
    "type": 2
}
```

Example GET_CLIENTS response with annotation:

```yaml
{
    "category": 2,
    "sequence": 2,
    "type": 2,
    "data": [
        {
            "id": "Qzmz1eKw",                                   // internal id for this session
            "name": "CLI",                                      // optional name for this session
            "device": {                                         // client device
                "id": "paprika.lan",                            // device id
                "name": "My Desktop PC (paprika.lan)",          // device name with alias if provided
                "remote": false                                 // is server remote to this device?
            }
        },
        {
            "id": "VPWqipXE",
            "name": "Edbot Studio UI",
            "device": {
                "id": "paprika.lan",
                "name": "My Desktop PC (paprika.lan)",
                "remote": false
            }
        }
    ]
}
```

---

**GET_SERVERS**

Get all available servers. This call will initiate a LAN broadcast.

Example GET_SERVERS request:

```yaml
{
    "category": 1,
    "sequence": 3,
    "type": 3
}
```

Example GET_SERVERS response with annotation:

```yaml
{
    "category": 2,
    "sequence": 3,
    "type": 3,
    "data": [
        {
            "ip": "127.0.0.1",                                  // server ip address
            "port": "54255"                                     // server port number
        },
        {
            "ip": "192.168.1.27",
            "port": "54255"
        }
    ]
}
```

---

**GET_SENSORS**

Get the sensor values

---

**GET_SERVOS**

Get the servo values

---

**RUN_MOTION**

Run a motion

---

**SET_SERVOS**

Set servo parameters

---

**SET_SPEAKER**

Set the speaker parameters

---

**SET_DISPLAY**

Set the display parameters

---

**SET_OPTIONS**

Set connection specific options

---

**SET_CUSTOM**

Set a custom value

---

**SAY**

Speak the text

---

**RESET**

Reset to initial values

### Update & Delete Messages

These messages are sent from the server to indicate data has been added, updated or deleted.
They can be used to update the data received in the **INIT** response.

Example update message:

```json
{
    "category": 3,
    "data": {
        "robots": {
            "Anna": {
                "control": "pepper"
            }
        }
    }
}
```

Example delete message:

```json
{
    "category": 4,
    "data": {
        "path": "robots.Anna"
    }
}
```

## Licence

Distributed under the MIT Licence. See the [LICENCE](../main/LICENCE) for more information.
