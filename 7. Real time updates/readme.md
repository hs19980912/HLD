**Real Time Updates**
  - Server-To-Client
    1. **Polling**
    2. **Long Polling**
    3. **SSE**
        - SSE uses the text/event-stream MIME type and chunked transfer encoding to stream events from server to client.
        - $EventSource$ API at Client's side.
        - HTTP headers:
          - Content-Type: text/event-stream
          - Connection: keep-alive
          - Cache-Control: no-cache
        - Works great with L7 LB and needs sticky sessions.
        - Reconnection: 
          - EventSource object explicitly handles the reconnection logic.
          - "Last event ID" received can is used at the client side for at-least-once-guarantee delivery.
        - Infra:
          - Each server instance holds its own active SSE connections.
          - Real systems scale SSE with Redis Pub/Sub.
          - 
    4. **Web Sockets**
        - Full-Duplex nature and highly performant(since we reduce the http overload).
        - Works on $upgrade$ protocol. After initial HTTP handshake, the protocol gets converted to L4, purely TCP based.
        - Needs L4 LB with sticky sessions.
        - Statefulness introduces a lot of challenges in deployments and Scaling.
        - Using a "least connections" strategy at the load balancer helps distribute WebSocket clients more evenly across endpoint servers.
    5. **Push notifications**
    6. **WebRTC**
  - Inter Server communication
    - Server using Pub/Sub to to listen to updates.


**Misc:**
  - HTTP/2 Multiplexing (Application level), and TCP multiplexing (Transport level)
  - HTTP/2 fixes application-level head-of-line blocking, But TCP still causes transport-level head-of-line blocking
    - History:
      - HTTP/1.0 chose to close after each request
      - HTTP/1.1 added keep-alive
      - HTTP/2 required a persistent TCP connection
  - HTTP/2 is still 100% compliant with HTTP spec
    - every request has its own stream ID, HEADERS frame, DATA frames, END_STREAM flag
  - How Chrome Manages Streams Inside HTTP/2
  - Why HTTP/2 Multiplexing Is Powerful
    - Without HTTP/2 (HTTP/1.1):
        - Browser opens max 6 connections per domain
        - SSE consumes 1 full TCP connection
        - Loading more data competes for connections
    - With HTTP/2:
        - Browser opens 1 TCP connection total
        - Images, CSS, JS, API calls, SSE all happen in parallel
        - No head-of-line blocking (HOLB) at HTTP layer
        - One SSE stream doesn’t block other requests




### What makes SSE better than Long polling:
```
We already had Long polling. We also saw that in modern systems, very efficient type of long polling 
has been developed which do not cause any thread overload in the server side an also prevent busy waiting 
in the server side. 

So what inherent benefits does SSE provide over the long polling that long polling cannot do?
```

- **SSE Removes the Repeated Connection Overhead (Long Polling Cannot)**
  - Long Polling: Every time an update happens (or timeout), the server must close the connection. Client must create a new HTTP request + new TCP connection.
  - SSE: ONE TCP connection stays open for the entire session. No repeated handshakes, No repeated HTTP headers. SSE Uses Much Less Bandwidth vs Long Polling.
  - SSE Reduces Latency to Milliseconds.
- **SSE Provides True Streaming, SSE Is Far More Efficient for High-Frequency Updates**
  - Long polling = one response per request
  - SSE = multiple responses over one request
- **SSE Has Built-in Auto-Reconnection (Long Polling Does Not)**
- **SSE Supports Event IDs & Re-delivery (Long Polling Does Not)**
  - Guaranteed-at-least-once delivery is much easier in SSE.
- **SSE Plays Perfectly With HTTP/2**
  - HTTP/2 allows multiplexing multiple SSE streams over one TCP connection. Long polling cannot use HTTP/2 effectively because Requests need to be closed and new streams must be opened each time. Many SSE streams can be sent over the same TCP connection.
    - Use case: Browser opens ONE TCP connection to server and can have SSE stream, GraphQL requests, Image loads, JS bundles, API calls etc. All over the same connection simultaneously.

| Feature                           | Long Polling | SSE                   |
| --------------------------------- | ------------ | --------------------- |
| One persistent connection         | ❌ No         | ✔ Yes                 |
| True real-time streaming          | ❌ No         | ✔ Yes                 |
| Auto reconnect                    | ❌ No         | ✔ Yes                 |
| Event replay                      | ❌ No         | ✔ Yes                 |
| Low bandwidth                     | ❌ No         | ✔ Yes                 |
| Low latency                       | ⚠ Almost     | ✔ True                |
| Works with high-frequency updates | ❌ Struggles  | ✔ Perfect             |
| Uses browser API                  | ❌ No         | ✔ Yes (`EventSource`) |






## **1. Polling — The Basic Approach**

> Client repeatedly sends HTTP requests at regular intervals asking:
> **“Do you have any updates?”**
> Server responds immediately — even if no new data exists.

### **Flow of Polling**

```
Client → GET /updates  → Server → “No updates”
(wait 5 seconds)
Client → GET /updates  → Server → “No updates”
(wait 5 seconds)
Client → GET /updates  → Server → “YES, update!”
```

### **Example Client Code**

```javascript
setInterval(() => {
  fetch("/updates")
    .then(res => res.json())
    .then(data => console.log("Update:", data));
}, 5000); // Every 5 sec
```

---

### **Pros & Cons of Polling**

| Pros                    | Cons                               |
| ----------------------- | ---------------------------------- |
| Very easy to implement  | Wastes bandwidth & CPU             |
| Works in all browsers   | High latency (depends on interval) |
| No special server needs | Bad scalability                    |
| Good for small apps     | Not truly real-time                |


### **Suitable When:**

* Data changes rarely  
* Real-time is **not critical**  
* Few users  
* Simple system (e.g., weather updates, dashboard refresh)   


## **2. Long Polling — Smarter Version**

> Client sends a request →  
> **Server HOLDS request open** until new data/event happens.  
> If data comes → server responds immediately.  
> If no data → server times out & closes connection.  
> Client immediately reconnects.  


### **Flow of Long Polling**

```
Client → GET /updates   → Server waits…
┌─────────────────────────────────────┐
│  If NEW data available → respond    │
│  If TIMEOUT → respond with empty    │
└─────────────────────────────────────┘
Connection closes → Client sends a NEW request
```



### **Client Code — Long Polling**

```javascript
function longPoll() {
    fetch("/updates")
      .then(res => res.json())
      .then(data => {
          console.log("Received:", data);
          longPoll();  // send again after response
      })
      .catch(() => setTimeout(longPoll, 2000)); // retry on error
}
longPoll(); // Start
```


### **Server Logic (Pseudo)**

```python
def handle_request(user):
    start_time = now()

    while True:
        if new_data_available(user):
            return data  # immediately respond

        if now() - start_time > 30 seconds:
            return timeout_response  # no data → close connection

        sleep(100 ms)  # avoid CPU spin
```


### **Pros & Cons of Long Polling**

| Pros                        | Cons                                                 |
| --------------------------- | ---------------------------------------------------- |
| Feels real-time             | Still uses HTTP overhead                             |
| Less bandwidth than polling | Many connections → needs async / event-driven server |
| Works in any browser        | Timeout → reconnection required                      |
| Easy fallback for real-time | Bad choice for huge traffic                          |

---

### **Socket Behavior**

| Aspect              | Polling | Long Polling                    |
| ------------------- | ------- | ------------------------------- |
| Socket lifetime     | Short   | Short (but held longer)         |
| Socket per update   | YES     | YES (new socket after response) |
| Same socket reused? | ❌ No    | ❌ No                            |
| Needs OS support?   | Normal  | Better with epoll/kqueue        |
| Timeout?            | No need | YES (required)                  |


## **3. Server-Sent Events (SSE)**

###  **What are SSEs?**

> A **one-way real-time communication** technique where the **server continuously pushes updates** to the **client over a single HTTP connection**, without requiring reconnection.


### **How It Works (Conceptually)**

```
Client → opens HTTP request → Server keeps connection open
Server → sends multiple events over time on SAME connection
Client → listens via EventSource API
```

* Only one HTTP request   
* Socket remains open   
* Server pushes data whenever available  
* Client receives updates **without polling**   



### **Key HTTP Headers**

```http
Content-Type: text/event-stream
Connection: keep-alive
Cache-Control: no-cache
```

These headers **tell the browser**:

> “This is a streaming response — don’t close the connection.”


### **Client-Side Code (Browser API)**

```javascript
const es = new EventSource("/events");
es.onmessage = (e) => console.log("Update:", e.data);
```

* Native to browser — NO library needed  
* Automatically reconnects if disconnected  
* Handles message formatting for you  


### **Server-Side Example (Node.js)**

```javascript
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  setInterval(() => {
    res.write(`data: ${new Date().toISOString()}\n\n`);
  }, 2000);
});
```

* Keeps connection open   
* Sends multiple messages   
* No need to reconnect   


### **SSE vs Long Polling vs WebSockets**

| Feature                        | Polling         | Long Polling                    | SSE             | WebSockets |
| ------------------------------ | --------------- | ------------------------------- | --------------- | ---------- |
| Stream over **one connection** | ❌ No            | ❌ No                            | ✔ Yes           | ✔ Yes      |
| Direction                      | Client → Server | Server → Client (after request) | Server → Client | Both ways  |
| Reconnection                   | Manual          | Manual                          | Automatic       | Manual     |
| Binary data support            | ❌               | ❌                               | ❌               | ✔          |
| For chat/games?                | No              | Maybe                           | No              | ✔ Yes      |
| For dashboards, stocks?        | No              | Maybe                           | ✔ Yes           | ✔ Yes      |


### **When to Use SSE**

* Live dashboards (analytics, stock prices)  
* Social media updates  
* Notifications system  
* CI/CD live logs  
* Sensor data streams  
* Server health monitoring  

**Not suitable for:**
* Chat applications
* Multiplayer gaming
* Real-time collaboration tools
  ➡ Use **WebSockets** instead.


> **SSE is one-way real-time streaming over a single persistent HTTP connection, ideal for server → client live updates, without the overhead of repeated polling or full WebSocket complexity.**

