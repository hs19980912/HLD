### Why is HTTP called Stateless and a web socket called stateful?
    HTTP is called "stateless" because each request from a client to a server is treated as an independent transaction. This means that the server does not retain any information about previous requests. Each interaction is separate, so the server doesn't need to remember anything about the client or their past requests.

    On the other hand, WebSockets are considered "stateful" because they establish a persistent connection between the client and the server. Once this connection is made, both parties can send and receive messages at any time without needing to re-establish the connection. This allows for continuous communication and the ability to maintain context, meaning the server can remember the state of the conversation or session.

In summary:

    HTTP (Stateless): Each request is independent; no memory of 
    previous interactions.
    WebSockets (Stateful): Persistent connection allows for ongoing communication and context retention.    
#
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
##
### Why is separate database for each microservice the preferred way over single DB for the entire application?
    Using a separate database for each microservice offers several advantages over a single database approach:

1. **Decoupling**: Each microservice operates independently, allowing teams to develop, deploy, and scale them without affecting others. This autonomy minimizes the risk of changes in one service impacting others.

2. **Tailored Data Models**: Different services often have distinct data requirements. Separate databases allow each microservice to use the most suitable data model and database technology (SQL, NoSQL, etc.) for its needs.

3. **Scalability**: Independent databases can be scaled individually based on the specific needs of each microservice, enabling more efficient resource allocation.

4. **Fault Isolation**: If one service's database encounters issues, it doesn’t necessarily compromise the entire application. This enhances overall system reliability.

5. **Data Governance and Security**: Each service can implement its own security policies, access controls, and data governance practices, allowing for better compliance and data management.

6. **Independent Deployment**: With separate databases, microservices can be deployed independently, facilitating continuous delivery and integration practices.

7. **Easier Technology Upgrades**: Teams can update or change their databases without coordinating with others, enabling more flexibility in adopting new technologies.

8. **Clear Ownership**: Separate databases establish clear ownership of data within teams, fostering accountability and focused expertise.

Overall, this approach aligns with the microservices architecture's core principles of autonomy and flexibility, leading to a more maintainable and resilient system.
#
