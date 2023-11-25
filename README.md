#### Java SE 6+/Android 4.1+ WebSocket client/server package,  MIT (c) 2020-2023 miktim@mail.ru<br/>
#### Release notes:

- small and easy;  
- Java SE 6+/Android 4.1+ compatible (see WebSocket-Android-Test repo:  
  [https://github.com/miktim/WebSocket-Android-Test](https://github.com/miktim/WebSocket-Android-Test) ).  
      The ./dist/websocket jar file was built using JDK 1.8.0 with the target JDK 6;  
- in accordance with RFC6455: [https://datatracker.ietf.org/doc/html/rfc6455/](https://datatracker.ietf.org/doc/html/rfc6455/) ;  
- supported WebSocket version: 13;  
- WebSocket extensions are not supported;  
- supports cleartext/TLS connections (without tunneling);  
- supports Internationalized Domain Names (IDNs); 
- incoming WebSocket messages are represented by input streams.

#### Examples:
Creating and running a Java server for TLS connections:  

	...
	// create server side connection handler
	WsConnection.EventHandler handler = new WsConnection.EventHandler() {
		@Override
		onOpen(WsConnection conn, String subProtocol) {
			conn.send("Hello, Client!");
			// ... do something
		}
		@Override
		onMessage(WsConnection conn, InputStream is, boolean isText) {
			if (isText) { // is UTF-8 text stream?
			// ... UTF-8 text do something
			} else {
			// ... binary data do something
			};
		}
		@Override
		onError(WsConnection conn, Throwable e) {
			// ... do something
		}
		@Override
		onClose(WsConnection conn, WsStatus status) {
			// ... do something
		}
	};
	try {
	// create WebSocket factory
		WebSocket webSocket = new WebSocket();
	// set Java style key store file
		webSocket.setKeyFile(new File("myKeyFile.jks"), "password");
	// create and start server with "dumb" server handler and default connection parameters
		WsServer server = webSocket.SecureServer(8443, handler, new WsParameters()).launch();	
	} (Exception e) {
		e.printStackTrace();
	}
	...
Creating a TLS connection for an Android client:  

	...
	// create client connection handler
	WsConnection.EventHandler handler = new WsConnection.EventHandler() {
		@Override
		onOpen(WsConnection conn, String subProtocol) {
			conn.send("Hello, Server!");
			// ... do something
		}
		@Override
		onMessage(WsConnection conn, InputStream is, boolean isText) {
			if (isText) { // is UTF-8 text stream?
			// ... UTF-8 text do something
			} else {
			// ... binary data do something
			};
		}
		@Override
		onError(WsConnection conn, Throwable e) {
			// ... do something
		}
		@Override
		onClose(WsConnection conn, WsStatus status) {
			// ... do something
		}
	};
	try {
	// create WebSocket factory
		WebSocket webSocket = new WebSocket();
	// set the trusted key store file in Android style
		webSocket.setKeyFile(new File("myKeyFile.bks"), "password");
	// create client TLS connection with default parameters
		WsConnection conn = webSocket.connect("wss://hostname:8443", handler, new WsParameters()).launch();	
	} (Exception e) {
		e.printStackTrace();
	}
	...
   
#### package org.miktim.websocket  
#### Overview:
- [Class WebSocket](#WebSocket);  
- [Class WsServer](#WsServer);  
- [Interface WsServer.EventHandler](#ServerHandler);  
- [Class WsConnection](#WsConnection);  
- [Interface WsConnection.EventHandler](#ConnectionHandler);  
- [Class WsParameters](#WsParameters);  
- [Class WsStatus](#WsStatus).

**<a id = "WebSocket">Class WebSocket:</a>**  
   Creator of servers and connections.
 
Constant:

- static String VERSION = "4.1.0";

Constructors:
  
- WebSocket() throws NoSuchAlgorithmException; // check if SHA1 exists  
- WebSocket(InetAddress bindAddr)
         throws SocketException, NoSuchAlgorithmException;  

Methods:

- static URI idnURI(String uri) throws URISyntaxException;  
  \- converts host name as defined by RFC3490 and creates URI  
  
- static void setKeyStore(String keyFilePath, String password);  
  \- sets system properties javax.net.ssl.keyStore/keyStorePassword  
- static void setTrustStore(String keyFilePath, String password);  
  \- sets system properties javax.net.ssl.trustStore / trustStorePassword

- void setKeyFile(File keyfile, String password);  
  \- sets keyStore file (server) or trustStore file (client)
- void resetKeyFile();

- WsServer Server(int port, WsConnection.EventHandler handler, WsParameters wsp)   throws IOException, GeneralSecurityException;  
  \- creates cleartext connections server
 
- WsServer SecureServer(int port, WsConnection.EventHandler handler, WsParameters wsp)   throws IOException, GeneralSecurityException;  
  \- creates TLS connections server  

Note: Servers are created using a "dumb" server handler. You can set your own handler or start the server immediately.

- WsConnection connect(String uri, WsConnection.EventHandler handler, WsParameters wsp) throws URISyntaxException, IOException, GeneralSecurityException;  
  \- creates and starts a client connection. URI's scheme (ws: | wss:) and host are required, user-info@ and #fragment - ignored

- InetAddress getBindAddress();
- WsServer[ ] listServers();  
  \- lists active servers.
- WsConnection[ ] listConnections();  
  \- lists active connections.  
- void closeAll();
- void closeAll(String closeReason);  

Note: Close functions closes all active servers/connections with code 1001 (GOING_AWAY) with or without close reason.

**<a id = "WsServer">Class WsServer extends Thread:</a>**  

   Methods:  
   
- WsServer setHandler(WsServer.EventHandler handler);  
- void start(); // start server  
- WsServer launch(); // start server  
- boolean isOpen();  
- boolean isInterrupted(); // ServerSocket error or interrupt  
- boolean isSecure();  
- int getPort();  
        - returns the port number on which this socket is listening.  
- InetAddress getBindAddress();  
        - returns ServerSocket bind address or null  
- WsParameters getParameters();  
- WsConnection[] listConnections();  
- void close();  
- void close(String closeReason);  
        - close methods also closes all active associated connections (code GOING_AWAY)  
- void interrupt();  
        - server interupted, associated connections stay alive and can be closed in the usual way


**<a id = "ServerHandler">Interface WsServer.EventHandler</a>:**
  
Methods:

- void onStart(WsServer server);
- boolean onAccept(WsServer server, WsConnection conn);  
        - called BEFORE WebSocket connection handshake;  
        - the returned value of true means approval of the connection,  
          the value of false means the closure of the client connection

- void onStop(WsServer server, Exception error);  
        - the error is null or a ServerSocket exception if occured


**<a id = "WsConnection">Class WsConnection extends Thread:</a>**
 
Constant:  
 
- MESSAGE\_QUEUE\_CAPACITY = 3  
        - queue overflow leads to an error and connection closure  
        
Methods:  

- void send(InputStream is, boolean isUTF8Text) throws IOException;   
        - is: input stream of binary data or UTF-8 encoded text  
- void send(String message) throws IOException;  // send text  
- void send(byte[ ] message) throws IOException; // send binary data
- void setHandler(WsConnection.EventHandler newHandler);  
        - calls onClose in the old handler, then calls onOpen in the new handler
- boolean isClientSide();  // returns true for   client connections  
- boolean isOpen();  // WebSocket connection is open
- boolean isSecure();   // is TLS connection
- WsConnection[ ] listConnections();
        // the client side returns only itself
- String getSSLSessionProtocol();
  // returns TLS protocol or null  
- WsStatus getStatus();  
  \- returns the connection status. The function is synchronized with the completion of the WebSocket connection handshake 
- String getSubProtocol();  
  \- returns handshaked WebSocket subprotocol or null 
- String getPeerHost(); // returns remote host name or null  
- int getPort(); // returns connected port  
- String getPath(); // returns http request path or null
- String getQuery(); // returns http request query or null  
- WsParameters getParameters(); // returns connection parameters
- void close();   
        - closes connection with status code 1005 (NO\_STATUS)
- void close(int statusCode, String reason);   
        - the status code outside 1000-4999 will be replaced with 1005 (NO\_STATUS),  
          the reason is ignored;  
        - a reason that is longer than 123 BYTES is truncated;  
        - the method blocks outgoing messages (sending methods throw IOException);  
        - isOpen() returns false;  
        - incoming messages are available until the closing handshake completed.  
        
**<a id = "ConnectionHandler">Interface WsConnection.EventHandler:</a>**  
   
There are two scenarios for handling events:  
        - onError - onClose, when the SSL/WebSocket handshake failed;  
        - onOpen - [onMessage - onMessage - ...] - [onError] - onClose.

Methods:
  
- void onOpen(WsConnection conn, String subProtocol);  
        - the second argument is the negotiated WebSocket subprotocol or null.
    
- void onMessage(WsConnection conn, InputStream is, boolean isUTF8Text);  
        - the message is an InputStream of binary data or UTF-8 encoded text;  
        - exiting the handler closes the stream (not connection!).  
- void onError(WsConnection conn, Throwable e);  
        - any exception closes the WebSocket connection;  
        - large incoming messages may throw an OutOfMemoryError;

- void onClose(WsConnection conn, WsStatus closeStatus);



**<a id = "WsParameters">Class WsParameters:</a>**  
WebSocket connection parameters  

Constructor:
 
- WsParameters();

Methods:

- WsParameters setSubProtocols(String[ ] subps);  
        - set WebSocket subProtocols in preferred order
- String[ ] getSubProtocols(); // null is default
- WsParameters setHandshakeSoTimeout(int millis);  
        - TLS and WebSocket open/close handshake timeout
- int getHandshakeSoTimeout();
- WsParameters setConnectionSoTimeout(int millis, boolean pingEnabled)  
        - data exchange timeout
* int getConnectionSoTimeout();
- boolean isPingEnabled(); // enabled by default
- WsParameters setPayloadBufferLength(int len); 
        - outgoing messages max payload length, min len = 125 bytes
- int getPayloadBufferLength();
- WsParameters setMaxMessageLength(int len); 
        - incoming messages max length. If exceeded, the connection 
          will be terminated with the MESSAGE_TOO_BIG status code
- int getMaxMessageLength(); // default: 1 MiB
- WsParameters setSSLParameters(SSLParameters sslParms);  
        - javax.net.ssl.SSLParameters used:
          Protocols, CipherSuites, NeedClientAut, WantClientAuth
- SSLParameters getSSLParameters();
- WsParameters setBacklog(int num);  
        - maximum number of pending connections on the ServerSocket 
- int getBacklog(); // default value is -1: system depended


**<a id = "WsStatus">Class WsStatus:</a>**  
WebSocket connection status

Parameters:

- int code;         // closing code (1000-4999)
- String reason;    // closing reason (max length 123 BYTES)
- boolean wasClean; // WebSocket closing handshake completed cleanly
- boolean remotely; // closed remotely
- Throwable error;  // closing exception or null

**Usage examples see in:**   
  ./test/websocket/WssConnectionTest.java   
  ./test/websocket/WsServerTest.java   
  ./test/websocket/WssClientTest.java
