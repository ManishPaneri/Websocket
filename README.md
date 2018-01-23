#Websocket
----

`WebSocketConfig` is annotated with `@Configuration` to indicate that it is a Spring configuration class.
It is also annotated {AtEnableWebSocketMessageBroker}[`@EnableWebSocketMessageBroker`].
As its name suggests, `@EnableWebSocketMessageBroker` enables WebSocket message handling, backed by a message broker.

The `configureMessageBroker()` method overrides the default method in `WebSocketMessageBrokerConfigurer` to configure the message broker.
It starts by calling `enableSimpleBroker()` to enable a simple memory-based message broker to carry the greeting messages back to the client on destinations prefixed with "/topic".
It also designates the "/app" prefix for messages that are bound for `@MessageMapping`-annotated methods.
This prefix will be used to define all the message mappings; for example, "/app/hello" is the endpoint that the `GreetingController.greeting()` method is mapped to handle.

The `registerStompEndpoints()` method registers the "/gs-guide-websocket" endpoint, enabling SockJS fallback options so that alternate transports may be used if WebSocket is not available. The SockJS client will attempt to connect to "/gs-guide-websocket" and use the best transport available (websocket, xhr-streaming, xhr-polling, etc).


== Create a browser client

With the server side pieces in place, now let's turn our attention to the JavaScript client that will send messages to and receive messages from the server side.

Create an index.html file that looks like this:

`src/main/resources/static/index.html`
[source,html]
----
include::complete/src/main/resources/static/index.html[]
----

This HTML file imports the `SockJS` and `STOMP` javascript libraries that will be used to communicate with our server
using STOMP over websocket. We're also importing here an `app.js` which contains the logic of our client application.

Let's create that file:

`src/main/resources/static/app.js`
[source,javascript]
----
include::complete/src/main/resources/static/app.js[]
----

The main piece of this JavaScript file to pay attention to is the `connect()` and `sendName()` functions.

The `connect()` function uses https://github.com/sockjs[SockJS] and {Stomp_JS}[stomp.js] to open a connection to "/gs-guide-websocket", which is where our SockJS server is waiting for connections. Upon a successful connection, the client subscribes to the "/topic/greetings" destination, where the server will publish greeting messages. When a greeting is received on that destination, it will append a paragraph element to the DOM to display the greeting message.

The `sendName()` function retrieves the name entered by the user and uses the STOMP client to send it to the "/app/hello" destination (where `GreetingController.greeting()` will receive it).

== Make the application executable

Although it is possible to package this service as a traditional link:/understanding/WAR[WAR] file for deployment to an external application server, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java `main()` method. Along the way, you use Spring's support for embedding the link:/understanding/Tomcat[Tomcat] servlet container as the HTTP runtime, instead of deploying to an external instance.


`src/main/java/hello/Application.java`
[source,java]
----
include::complete/src/main/java/hello/Application.java[]
----

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/master/spring-boot-application.adoc[]

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/master/build_an_executable_jar_subhead.adoc[]

include::https://raw.githubusercontent.com/spring-guides/getting-started-macros/master/build_an_executable_jar_with_both.adoc[]


Logging output is displayed. The service should be up and running within a few seconds.

== Test the service

Now that the service is running, point your browser at http://localhost:8080 and click the "Connect" button.

Upon opening a connection, you are asked for your name. Enter your name and click "Send". Your name is sent to the server as a JSON message over STOMP. After a 1-second simulated delay, the server sends a message back with a "Hello" greeting that is displayed on the page. At this point, you can send another name, or you can click the "Disconnect" button to close the connection.

