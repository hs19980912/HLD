## Proxy as a broader concept

A proxy is an intermediate component that sits between a client and a server. It acts as a middleman to process requests/responses on behalf of the client or the server.

### 🔁 Forward Proxy vs Reverse Proxy

| Feature              | **Forward Proxy**                            | **Reverse Proxy**                              |
|----------------------|-----------------------------------------------|-------------------------------------------------|
| **Who it serves**     | **Clients (users)**                          | **Servers (backend services)**                  |
| **Direction**         | Client → Forward Proxy → Server             | Client → Reverse Proxy → Server                |
| **Hides the...**      | **Client** from the server                   | **Server** from the client                      |
| **Common Use Cases**  | - Browsing anonymously <br> - Bypassing geo-blocks <br> - Content filtering in orgs | - Load balancing <br> - API gateways <br> - Caching <br> - SSL termination |
| **Example Tooling**   | Squid, Privoxy                                | NGINX, HAProxy, AWS ALB, Cloudflare             |

---

### 🧭 Forward Proxy — "Client's Representative"

- **Use case**: You're in a corporate network, and all internet traffic must go through a proxy.
- **Why**: Controls or monitors what clients access, sometimes anonymizes traffic.
- **Example**:
    > Your computer → Forward Proxy → Google.com

---

### 🛡️ Reverse Proxy — "Server's Bodyguard"

- **Use case**: You're accessing a website like YouTube, but your request hits Cloudflare first.
- **Why**: Handles load balancing, caching, and security before hitting the real backend.
- **Example**:
    > You → Cloudflare (Reverse Proxy) → YouTube backend

