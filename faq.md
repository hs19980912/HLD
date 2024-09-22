### Why is HTTP called Stateless and a web socket called stateful?
    HTTP is called "stateless" because each request from a client to a server is treated as an independent transaction. This means that the server does not retain any information about previous requests. Each interaction is separate, so the server doesn't need to remember anything about the client or their past requests.

    On the other hand, WebSockets are considered "stateful" because they establish a persistent connection between the client and the server. Once this connection is made, both parties can send and receive messages at any time without needing to re-establish the connection. This allows for continuous communication and the ability to maintain context, meaning the server can remember the state of the conversation or session.

In summary:

    HTTP (Stateless): Each request is independent; no memory of 
    previous interactions.
    WebSockets (Stateful): Persistent connection allows for ongoing communication and context retention.    

### Examples of Client Server architecture
    Client-server architecture is a computing model where client devices request services and resources from centralized servers. Here are some common examples:
    
1. Web Applications

    Example: Websites like Google, Facebook, or Amazon.
    How It Works: The browser (client) sends requests to a web server, which processes them and returns HTML, CSS, and JavaScript files.

2. Email Services

    Example: Gmail, Outlook.
    How It Works: The email client (like Outlook or a web browser) connects to an email server to send and receive messages using protocols like SMTP, IMAP, or POP3.

3. File Sharing Services

    Example: Dropbox, Google Drive.
    How It Works: The client application allows users to upload or download files from a centralized server where the files are stored.

4. Database Applications

    Example: A web application using a MySQL database.
    How It Works: The application server (client) communicates with the database server to perform operations like querying or updating data.

5. Online Gaming

    Example: Fortnite, World of Warcraft.
    How It Works: The game client connects to a game server that manages the game state, player interactions, and data synchronization.

6. Streaming Services

    Example: Netflix, Spotify.
    How It Works: The client (app or web browser) requests media content from the streaming server, which streams audio or video data.

7. Chat Applications

    Example: Slack, WhatsApp Web.
    How It Works: The client application connects to a chat server to send and receive messages in real-time.

8. Remote Desktop Services

    Example: Microsoft Remote Desktop, TeamViewer.
    How It Works: The client connects to a remote server, allowing users to access and control another computer remotely.