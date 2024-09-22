### Why is HTTP called Stateless and a web socket called stateful?
    HTTP is called "stateless" because each request from a client to a server is treated as an independent transaction. This means that the server does not retain any information about previous requests. Each interaction is separate, so the server doesn't need to remember anything about the client or their past requests.

    On the other hand, WebSockets are considered "stateful" because they establish a persistent connection between the client and the server. Once this connection is made, both parties can send and receive messages at any time without needing to re-establish the connection. This allows for continuous communication and the ability to maintain context, meaning the server can remember the state of the conversation or session.

In summary:

    HTTP (Stateless): Each request is independent; no memory of 
    previous interactions.
    WebSockets (Stateful): Persistent connection allows for ongoing communication and context retention.    


