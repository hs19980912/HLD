## POST API
    POST: /create-url
    params: long-url
    status code: 201 created

Here's an example of how that might look if you are building it or dealing with it in a RESTful API environment:

### Request
```
POST /create-url
Content-Type: application/json

{
  "long-url": "https://example.com/some/long/url"
}
```

### Response (status code: 201 Created)
```
{
  "short-url": "https://short.ly/abc123"
}
```

In this scenario:
- **POST**: You are sending data to the server (in this case, a long URL).
- **/create-url**: The endpoint where the URL shortening service is implemented.
- **Status code 201 (Created)**: Indicates that the request was successful and a new resource (shortened URL) was created.

### What does status code 201 mean? and what is the context here?
**Status code 201 (Created)** is part of the HTTP response status codes, and it specifically indicates that a **request has been successful** and, as a result, **a new resource has been created** on the server.

### Meaning of Status Code 201:
- **201 Created**: This means that the server successfully processed the request and created a new resource. 
- The response typically includes a **location header** or a body containing the newly created resource (like a shortened URL, in your example).
- It is used with **POST** requests when a client sends data to the server to create something (e.g., a new record, a file, or in your case, a shortened URL).

### Context of 201 in Your Example:
- In your example (`POST /create-url`), the client is requesting the server to take a **long URL** and generate a shortened version of it.
- After successfully creating the shortened URL, the server responds with a **201 Created** status, indicating that the process was successful.
- The response might contain the shortened URL in the body or via a **Location** header, allowing the client to know where the newly created resource is located.

So the context is that **a resource (shortened URL)** has been created as a result of the request to the `/create-url` endpoint.

## But as per my understanding server only sends the response in terms of standard status code, how come it send the shortened URL as the response?
Your understanding is partially correct, but let's clarify the situation.

When a server sends an HTTP status code like **201**, it's essentially communicating the **status** of the request. However, it's not limited to sending just the status code. The server can (and often does) send additional data along with the status code in the **response body** or headers.

In the case of a **201 Created** status, along with the status code, the server often includes relevant information about the newly created resource. This is part of the HTTP standard and is common in RESTful APIs.

### Response Components:
- **Status Code**: Indicates the result of the request (like 201 for created).
- **Response Body**: Can contain extra information, such as the details of the newly created resource (in your example, the shortened URL).
- **Response Headers**: May include additional metadata, such as the `Location` header pointing to the new resource's URL.

### Example:

#### Request:
```
POST /create-url
Content-Type: application/json

{
  "long-url": "https://example.com/some/long/url"
}
```

#### Response (status code 201):
```
Status: 201 Created
{
  "short-url": "https://short.ly/abc123"
}
```

In this case, the response includes:
1. **Status Code 201** to indicate that a resource has been created.
2. **Response Body** with the newly created shortened URL.

The body of the response is where additional information like the shortened URL can be sent. The server is free to send any extra data that helps the client process the request.

### Why Send Data with Status Code 201?
- The client needs to know the **newly created resource's identifier or location**, like the shortened URL.
- The response body helps the client use the new resource immediately without making another request.

## But where did we specify that the key must be ""short-url" in the response?
Great question! The key like `"short-url"` in the response is typically **defined by the API's design**. It's up to the **API developer** to decide how the response should be structured. This kind of detail is usually specified in the API documentation or agreement between the client and server.

Let's break it down:

### How Response Keys Are Specified:

1. **API Design**:
   - The API **developer defines** how the response will look, including the keys. In this case, the API could be designed to return the shortened URL under the key `"short-url"`.
   - If you're building or consuming an API, you would refer to the **API documentation** to know exactly what to expect in the response.

2. **API Documentation**:
   - The **documentation** for the API would specify the request and response structure, including expected keys like `"short-url"`.
   - Example documentation could say:
     - **Endpoint**: `POST /create-url`
     - **Request Body**:
       ```json
       {
         "long-url": "https://example.com"
       }
       ```
     - **Response Body**:
       ```json
       {
         "short-url": "https://short.ly/abc123"
       }
       ```

3. **Standardization**:
   - There’s no inherent rule that says the key has to be `"short-url"`. The key could be anything like `"url"`, `"shortenedLink"`, or even `"new-resource"`. It depends on how the API is designed.
   - The server and client must **agree** on the key names through documentation or convention.

### How You Would Define It:
If you were designing an API, you would specify this in the server-side code. For instance, if you're using a framework like **Express.js** (Node.js), you might write something like this in the server logic:

```js
app.post('/create-url', (req, res) => {
  const longUrl = req.body['long-url'];
  const shortUrl = shortenUrl(longUrl);  // your logic to shorten URL
  res.status(201).json({ 'short-url': shortUrl });
});
```

In this example, the **key** `"short-url"` is explicitly defined in the server code. When the client sends the request, the server responds with this key.

### Customization:
- If you're building the API, you have full control over the key names in the response.
- If you're consuming the API, the key names must match what's defined in the API documentation.

### Summary:
The key `"short-url"` doesn't come from any implicit rule—it’s defined by the API. Both the server and client must follow the agreed-upon structure, usually outlined in API documentation.

## What would the above code look like in Spring boot?
In a **Spring Boot** application, you can achieve the same result as the Express.js example by defining a REST controller to handle the `POST` request. Here's what the code would look like:

### Spring Boot Code Example:

#### 1. **Create a Model for the Request and Response**

We need a **DTO (Data Transfer Object)** for handling the incoming request (long URL) and the outgoing response (short URL).

```java
// Request DTO (for incoming long URL)
public class UrlRequest {
    private String longUrl;

    // Getters and Setters
    public String getLongUrl() {
        return longUrl;
    }

    public void setLongUrl(String longUrl) {
        this.longUrl = longUrl;
    }
}

// Response DTO (for outgoing short URL)
public class UrlResponse {
    private String shortUrl;

    public UrlResponse(String shortUrl) {
        this.shortUrl = shortUrl;
    }

    // Getter
    public String getShortUrl() {
        return shortUrl;
    }
}
```

#### 2. **Create a Service to Handle the URL Shortening Logic**

In this step, you would typically write your URL shortening logic. For simplicity, let's assume it generates a random string for the shortened URL.

```java
import org.springframework.stereotype.Service;

@Service
public class UrlShorteningService {

    public String shortenUrl(String longUrl) {
        // Here you can implement a real URL shortening algorithm
        // For now, let's just simulate it with a hardcoded string.
        return "https://short.ly/" + Math.abs(longUrl.hashCode());
    }
}
```

#### 3. **Create the REST Controller**

The controller will handle the **`POST /create-url`** request and return the appropriate response. We'll use the DTOs for input and output.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/create-url")
public class UrlShortenerController {

    @Autowired
    private UrlShorteningService urlShorteningService;

    @PostMapping
    public ResponseEntity<UrlResponse> createShortUrl(@RequestBody UrlRequest request) {
        // Call service to generate short URL
        String shortUrl = urlShorteningService.shortenUrl(request.getLongUrl());
        
        // Return 201 Created with the short URL in the response body
        UrlResponse response = new UrlResponse(shortUrl);
        return new ResponseEntity<>(response, HttpStatus.CREATED);
    }
}
```

### Breakdown:
1. **Request DTO** (`UrlRequest`): This object will handle the incoming JSON request body containing the long URL.
    ```json
    {
      "longUrl": "https://example.com/some/long/url"
    }
    ```
2. **Response DTO** (`UrlResponse`): This object will format the response with the key `"shortUrl"` containing the shortened URL.
    ```json
    {
      "shortUrl": "https://short.ly/abc123"
    }
    ```
3. **Service Layer** (`UrlShorteningService`): Contains the business logic to generate a shortened URL. You can implement the actual URL shortening logic here (e.g., generating a random hash or mapping long URLs to short ones).
4. **Controller** (`UrlShortenerController`): 
   - **`@PostMapping`**: Specifies that this method will handle **POST** requests.
   - The `ResponseEntity` is used to return the response along with the HTTP status code **201 Created**.

### Request Example:
When the client sends a `POST` request like this:

```
POST /create-url
Content-Type: application/json

{
  "longUrl": "https://example.com/some/long/url"
}
```

### Response Example (with 201 status code):
The server will respond with a **201 Created** status and the shortened URL:

```json
{
  "shortUrl": "https://short.ly/abc123"
}
```

### Notes:
- **`@Autowired`** is used to inject the `UrlShorteningService` into the controller.
- **`ResponseEntity<UrlResponse>`** is used to wrap the response object and send an appropriate HTTP status code (**201 Created**).

This approach is common in **Spring Boot** to create REST APIs.

## Is there a way we can dynamically change the response json's key from shortUrl to something else with each request?
Yes, you can **dynamically change the JSON response key** in Spring Boot. One way to achieve this is by using a `Map<String, String>` instead of a fixed response DTO (like `UrlResponse`). By using a `Map`, you can dynamically set the key-value pairs in the response based on the input or logic for each request.

Here's how you can implement it:

### Steps to Dynamically Change the JSON Key

1. **Modify the Controller to Accept a Key**:
   You can modify the request to include a key that the client wants to use for the shortened URL in the response.

2. **Use a Map for Dynamic Keys**:
   Instead of using a predefined DTO like `UrlResponse`, you can construct a `Map<String, String>` for the response body, which allows you to dynamically assign the key.

### Updated Code:

#### 1. **Modify the Request DTO**:
   Add a field for the dynamic key in the request body.

```java
public class UrlRequest {
    private String longUrl;
    private String responseKey;  // New field for dynamic key

    // Getters and Setters
    public String getLongUrl() {
        return longUrl;
    }

    public void setLongUrl(String longUrl) {
        this.longUrl = longUrl;
    }

    public String getResponseKey() {
        return responseKey;
    }

    public void setResponseKey(String responseKey) {
        this.responseKey = responseKey;
    }
}
```

#### 2. **Modify the Controller**:
   Instead of using a predefined DTO for the response, use a `Map<String, String>` to create a response with a dynamic key.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/create-url")
public class UrlShortenerController {

    @Autowired
    private UrlShorteningService urlShorteningService;

    @PostMapping
    public ResponseEntity<Map<String, String>> createShortUrl(@RequestBody UrlRequest request) {
        // Shorten the URL using the service
        String shortUrl = urlShorteningService.shortenUrl(request.getLongUrl());

        // Create a map to hold the dynamic response
        Map<String, String> response = new HashMap<>();
        
        // Dynamically set the key for the short URL, default to "shortUrl" if not provided
        String responseKey = (request.getResponseKey() != null) ? request.getResponseKey() : "shortUrl";
        response.put(responseKey, shortUrl);

        // Return the response with dynamic key and 201 Created status
        return new ResponseEntity<>(response, HttpStatus.CREATED);
    }
}
```

### Key Changes:
1. **Dynamic Key (`responseKey`)**:
   - The request body now contains a field called `responseKey`, which allows the client to specify the key for the shortened URL.
   - If the `responseKey` is not provided in the request, the server defaults to using `"shortUrl"`.

2. **Map for Dynamic Response**:
   - The `Map<String, String>` allows us to dynamically create a response with any key-value pair based on the client’s input.
   
### Request Example:

#### 1. Default Request (No Custom Key):
```json
POST /create-url
Content-Type: application/json

{
  "longUrl": "https://example.com/some/long/url"
}
```

#### Response with Default Key (`shortUrl`):
```json
{
  "shortUrl": "https://short.ly/abc123"
}
```

#### 2. Request with Custom Key:
```json
POST /create-url
Content-Type: application/json

{
  "longUrl": "https://example.com/some/long/url",
  "responseKey": "customKey"
}
```

#### Response with Custom Key (`customKey`):
```json
{
  "customKey": "https://short.ly/abc123"
}
```

### Explanation:
- In the request, the client can include the `responseKey` field to specify how they want the shortened URL to be labeled in the response JSON.
- If no `responseKey` is provided, the controller uses a default key (`"shortUrl"`).
- This approach provides the flexibility to dynamically generate responses with different keys based on the client's input.

### Summary:
- By using a **`Map<String, String>`** in the response, you can dynamically change the key based on the request.
- The key can be passed as part of the request (e.g., `responseKey`), and the controller logic will insert the appropriate key-value pair into the response.
