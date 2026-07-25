## Dropbox

1. **Upload Service**:
    - Introduce an Upload Service instead of exposing storage details to the client.
    - Using pre-signed URLs so the application servers don't become the data path is a good architectural decision.
    - Separating metadata from blob storage is also a common pattern.