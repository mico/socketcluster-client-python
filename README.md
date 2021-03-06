# asyncio version of socketcluster-client-python
Refer examples for more details :

Overview
--------
This client provides following functionality

- Easy to setup and use
- Can be used for extensive unit-testing of all server side functions
- Support for emitting and listening to remote events
- Automatic reconnection
- Pub/sub
- Authentication (JWT)
- python3.x.x

To install use
```python
    sudo pip install socketclusterclient
```

Description
-----------
Create instance of `Socket` class by passing url of socketcluster-server end-point

```python
    //Create a socket instance
    socket = Socketcluster.socket("ws://localhost:8000/socketcluster/")

```
**Important Note** : Default url to socketcluster end-point is always *ws://somedomainname.com/socketcluster/*.

#### Registering basic listeners

Different functions are given as an argument to register listeners

```python
        from socketclusterclient import Socketcluster
        import logging

        logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)


        def onconnect(socket):
            logging.info("on connect got called")


        def ondisconnect(socket):
            logging.info("on disconnect got called")


        def onConnectError(socket, error):
            logging.info("On connect error got called")


        def onSetAuthentication(socket, token):
            logging.info("Token received " + token)
            socket.setAuthtoken(token)


        def onAuthentication(socket, isauthenticated):
            logging.info("Authenticated is " + str(isauthenticated))


        if __name__ == "__main__":
            socket = Socketcluster.socket("ws://localhost:8000/socketcluster/")
            socket.setBasicListener(onconnect, ondisconnect, onConnectError)
            socket.setAuthenticationListener(onSetAuthentication, onAuthentication)
            socket.connect()
```

#### Connecting to server

- For connecting to server:

```python
    //This will send websocket handshake request to socketcluster-server
    socket.connect();
```

- By default reconnection to server is enabled , to configure delay for connection

```python
    //This will set automatic-reconnection to server with delay of 2 seconds and repeating it for infinitely
    socket.setdelay(2)
    socket.connect();
```

- To disable reconnection :

```python
   socket.setreconnection(False)
```

Emitting and listening to events
--------------------------------
#### Event emitter

- eventname is name of event and message can be String, boolean, int or JSON-object

```python

    socket.emit(eventname,message);

    # socket.emit("chat", "Hi")
```

- To send event with acknowledgement

```python

    socket.emitack("chat", "Hi", ack)  

    def ack(eventname, error, object):
        print "Got ack data " + object + " and error " + error + " and eventname is " + eventname
```

#### Event Listener

- For listening to events :

The object received can be String, Boolean, Long or JSONObject.

```python
     # Receiver code without sending acknowledgement back
     socket.on("ping", message)

     def message(eventname, object):
         print "Got data " + object + " from eventname " + eventname
```

- To send acknowledgement back to server

```python
    # Receiver code with ack
    socket.onack("ping", messsageack)

    def messsageack(eventname, object, ackmessage):
        print "Got data " + object + " from eventname " + eventname
        ackmessage("this is error", "this is data")

```

Implementing Pub-Sub via channels
---------------------------------

#### Creating channel

- For creating and subscribing to channels:

```python

    # without acknowledgement
    socket.subscribe('yell')

    #with acknowledgement
    socket.subscribeack('yell', suback)

    def suback(channel, error, object):
        if error is '':
            print "Subscribed successfully to channel " + channel
```

- For getting list of created channels :

```python
        channels = socket.getsubscribedchannels()

```





#### Publishing event on channel

- For publishing event :

```python

       # without acknowledgement
       socket.publish('yell', 'Hi dudies')

       #with acknowledgement
       socket.publishack('yell', 'Hi dudies', puback)

       def puback(channel, error, object):
           if error is '':
               print "Publish sent successfully to channel " + channel
```

#### Listening to channel

- For listening to channel event :

```python

        socket.onchannel('yell', channelmessage)

        def channelmessage(key, object):
            print "Got data " + object + " from key " + key

```

#### Un-subscribing to channel

```python
         # without acknowledgement
         socket.unsubscribe('yell')

         # with acknowledgement
         socket.unsubscribeack('yell', unsuback)

         def unsuback(channel, error, object):
              if error is '':
                   print "Unsubscribed to channel " + channel
```

#### Disable SSL Certificate Verification

```python
        socket = Socketcluster.socket("wss://localhost:8000/socketcluster/")
        socket.connect(sslopt={"cert_reqs": ssl.CERT_NONE})
```

#### HTTP proxy

Support websocket access via http proxy. The proxy server must allow "CONNECT" method to websocket port. Default squid setting is "ALLOWED TO CONNECT ONLY HTTPS PORT".

```python
        socket = Socketcluster.socket("wss://localhost:8000/socketcluster/")
        socket.connect(http_proxy_host="proxy_host_name", http_proxy_port=3128)
```
