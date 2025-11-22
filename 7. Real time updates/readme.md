1. Polling
2. Long Polling
3. SSE
4. Web Sockets




## **1. Polling — The Basic Approach**

###  **Definition**

> Client repeatedly sends HTTP requests at regular intervals asking:
> **“Do you have any updates?”**
> Server responds immediately — even if no new data exists.

---

### **Flow of Polling**

```
Client → GET /updates  → Server → “No updates”
(wait 5 seconds)
Client → GET /updates  → Server → “No updates”
(wait 5 seconds)
Client → GET /updates  → Server → “YES, update!”
```

---

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

---

### **Suitable When:**

* Data changes rarely  
* Real-time is **not critical**  
* Few users  
* Simple system (e.g., weather updates, dashboard refresh)   



## **2. Long Polling — Smarter Version**

### **Definition**

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

### **Polling vs Long Polling — Comparison Table**

| Feature               | Polling             | Long Polling                    |
| --------------------- | ------------------- | ------------------------------- |
| HTTP Connection       | Short-lived         | Short-lived but waits           |
| Latency               | Depends on interval | Lower                           |
| Updates when no data? | YES (useless)       | NO (waits)                      |
| Server load           | High                | Moderate                        |
| True real-time?       | ❌                   | Semi-real-time                  |
| Complexity            | Low                 | Medium                          |
| Good for many users?  | ❌ No                | ⚠ Depends (needs async servers) |
| Best For              | Rare updates        | Chat apps, notifications        |

---

### **Socket Behavior**

| Aspect              | Polling | Long Polling                    |
| ------------------- | ------- | ------------------------------- |
| Socket lifetime     | Short   | Short (but held longer)         |
| Socket per update   | YES     | YES (new socket after response) |
| Same socket reused? | ❌ No    | ❌ No                            |
| Needs OS support?   | Normal  | Better with epoll/kqueue        |
| Timeout?            | No need | YES (required)                  |

---

Here’s a **crisp and complete summary** of **Server-Sent Events (SSE)** — perfect for interview answers and deep understanding:


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

---

## **Final One-Line Summary**

> **SSE is one-way real-time streaming over a single persistent HTTP connection, ideal for server → client live updates, without the overhead of repeated polling or full WebSocket complexity.**

