# Lightstreamer - Quickstart Example - HTTP clients #
<!-- START DESCRIPTION lightstreamer-example-quickstart-client-socket -->

In this tutorial, we'll show a client that communicates with the Lightstreamer Server through **direct HTTP requests**, instead of leveraging one of the available Client Libraries.

The rationale for this is to enable the development of clients based on any technology. This way, it is possible receive real-time updates from Lightstreamer Server from programs written in **C, PHP, Ruby,** or any other language that allows **direct HTTP requests**.

This tutorial provides instructions to interact with the [Lightstreamer - Basic Chat Demo - Java Adapter](https://github.com/Lightstreamer/Lightstreamer-example-Chat-adapter-java).

## Details

This tutorial shows how you can open a streaming connection to Lightstreamer Server and interact with the Chat Demo Adapter Set.
To this purpose, it leans on the public installation of Lightstreamer Server on our demo site.

Its workflow is relatively simple:
* Connection and session creation.
* Subscription to an item on a control connection.
* Send a message.
* Receive the message echo.

*Note: When you send a message through our online Server, your IP address is publicly displayed.*

The interaction leverages Lightstreamer protocol for generic clients, known as TLCP.
This protocol is available for both HTTP-request-based and WebSocket transport.
In this tutorial, only the HTTP case is shown, leaning on the **cURL** tool for the manual submission of HTTP requests.
**cURL** is a widespread command-line utility used to test HTTP requests and responses. It is available for most operating systems and can be downloaded at: [https://curl.haxx.se](https://curl.haxx.se)

<!-- END DESCRIPTION lightstreamer-example-quickstart-client-socket -->

## Install
If you want to replicate this tutorial pointing to your local Lightstreamer Server, follow these steps:
* As prerequisite, the [Basic Chat Demo - Java Adapter](https://github.com/Lightstreamer/Lightstreamer-example-Chat-adapter-java) has to be deployed on your local Lightstreamer Server instance. Please check out that project and follow the installation instructions provided with it until you have your local Lightstreamer Server running.
* In the instructions below, replace the "push.lightstreamer.com" address with "localhost:8080".
* Also, in the instructions below, replace the "DEMO" Adapter Set name with "CHAT".

### Run the Demo

**Session Creation and Subscription to an Item**

First of all, let’s connect and create a session. Our request needs to specify:
* The protocol: LS_protocol=TLCP-2.0.0
* The client identifier: LS_cid=mgQkwtwdysogQz2BJ4Ji%20kOj2Bg
* The Adapter Set “DEMO”: LS_adapter_set=DEMO
* Finally, the host and path: http://push.lightstreamer.com/lightstreamer/create_session.txt

Here is the complete call with cURL; open a command or a shell window and type:
```cmd
curl -v -N -X POST -d "LS_adapter_set=DEMO&LS_cid=mgQkwtwdysogQz2BJ4Ji%20kOj2Bg" http://push.lightstreamer.com/lightstreamer/create_session.txt?LS_protocol=TLCP-2.0.0
```
Note that it is a single line.

The response from the Server looks like the following:
```cmd
CONOK,Sa5268cc2d401967bT4311553,50000,5000,*
SERVNAME,Lightstreamer HTTP Server
CLIENTIP,0:0:0:0:0:0:0:1
NOOP,sending placeholder data
[…]
NOOP,sending placeholder data
CONS,unlimited
PROBE
PROBE
PROBE
[…]
```

The response contains:
* The CONOK response, with the session ID, request limit (50000), keep alive time (5000), and control link (not defined).
* The SERVNAME notification, with the name of the Server.
* The CLIENTIP notification, with the client IP.
* Some NOOP notifications (the majority are omitted), needed to fill up the receive buffer of the client.
* The CONS notification, with the maximum bandwidth allowed (unlimited means no limit).
* Finally, repeating every few seconds, a PROBE notification to keep the stream connection alive.

You have to extract the session ID from the received response.

Now we will subscribe to an item that is supplied by the CHAT_ROOM Data Adapter. This adapter provides the following:
* A single item named “chat_room” that simulates a simple chat feed.
* Field names are the following: “timestamp”, “message”, “IP”, “nick”.
* For this data model the appropriate subscription mode is “DISTINCT”.

The previous call with cURL opened the stream connection and must be left intact to receive some data.
So, on a separate command line, let’s subscribe to an item using a request with the following characteristics:
* The operation is a subscription: LS_op=add
* The subscription ID to identify the subscription later.
* The Data Adapter “CHAT_ROOM”: LS_data_adapter=CHAT_ROOM
* The group is simply the name of an item: LS_group=chat_room
* The schema is composed by juxtaposing some field names: LS_schema=timestamp message
* The session ID is the one reported on the session creation response.
* The request ID to be able to match the corresponding response.
* Finally, the path is that of a control request: /lightstreamer/control.txt

Here is the complete call with cURL:
```cmd
curl -v -N -X POST -d "LS_op=add&LS_subId=1&LS_data_adapter=CHAT_ROOM&LS_group=chat_room&LS_schema=timestamp%20message&LS_mode=DISTINCT&LS_session=__<session-ID>__&LS_reqId=1" http://push.lightstreamer.com/lightstreamer/control.txt?LS_protocol=TLCP-2.0.0
```
Recall that it is a single line. Put the session ID of your session where appropriate (bold part).

The response from the Server is:
```cmd
REQOK,1
```
With 1 being the request ID. On the other command-line, where the stream connection is running, some new notifications appear:
```cmd
[…]
PROBE
SYNC,14
SUBOK,1,1,2
CONF,1,unlimited,filtered
PROBE
[…]
```
The meaning being:
* The SYNC notification reports the elapsed time on the server (in seconds).
* The SUBOK notification reports that the subscription with ID 1 has been successfully activated.
* That CONF notification reports that the subscription with ID 1 has no frequency limit and is filtered.
These notifications tell us that the item has been subscribed successfully.

**Send a Message and Receive its Echo**

To send a message to the Chat Demo Adapters we need a request with the following characteristics:
* The content of the message is just a well-known international salute: LS_message=CHAT|Ciao where the “CHAT|” part is just a second-level syntax in use by this specific Metadata Adapter.
* The progressive number of the message: LS_msg_prog=1
* The session ID is the one reported on the session creation response.
* The request ID to be able to match the corresponding response.
* Finally, the path is that of a message send request: /lightstreamer/msg.txt

Here is the complete call with cURL; on the second command line type:
```cmd
curl -v -N -X POST -d "LS_session=__<session-ID>__&LS_message=CHAT|Ciao&LS_msg_prog=1&LS_reqId=2" http://push.lightstreamer.com/lightstreamer/msg.txt?LS_protocol=TLCP-2.0.0
```
Recall that it is a single line. Put the session ID of your session where appropriate (bold part).

The response from the Server is:
```cmd
REQOK,2
```
In the case of messages, this standard response also acts as an acknowledge from the Server, meaning that the message has been correctly received and enqueued for processing.

On the other command-line, where the stream connection is running, some new notifications appear:
```cmd
[…]
PROBE
SYNC,37
MSGDONE,*,1
U,1,1|13:07:00|Ciao
PROBE
[…]
```
The meaning being:
* The MSGDONE notification tells that the message has been processed by the Server and delivered to the Metadata Adapter. Its arguments are the sequence of the message (we did not specify it, and hence it is an asterisk “*”) and the progressive number of the message.
* The real-time update reports the message as part of the chat feed, with the two fields being the timestamp and the message content.

The real-time update is, in fact, the echo of our previous message.

## See Also

### Lightstreamer Adapters Needed by This Client
<!-- START RELATED_ENTRIES -->

* [Lightstreamer - Basic Chat Demo - Java Adapter](https://github.com/Lightstreamer/Lightstreamer-example-Chat-adapter-java)
* [Lightstreamer - Reusable Metadata Adapters - Java Adapter](https://github.com/Lightstreamer/Lightstreamer-example-ReusableMetadata-adapter-java)

<!-- END RELATED_ENTRIES -->

## Lightstreamer Compatibility Notes

- Compatible with Lightstreamer SDK for Generic Clients version (i.e. TLCP Protocol) 2.0.0 or newer.

