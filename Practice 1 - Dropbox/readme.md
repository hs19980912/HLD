## Dropbox

- **Upload Service**:
    - Introduce an Upload Service instead of exposing storage details to the client.
    - Using pre-signed URLs so the application servers don't become the data path is a good architectural decision.
    - Separating metadata from blob storage is also a common pattern.

#### Multipart Upload Workflow:
1. Chunking and Hashing (client initiated)
    - The client is never allowed to dictate the number of chunks. If it did, a malicious user could request 10,000 Presigned URLs.
2.  
