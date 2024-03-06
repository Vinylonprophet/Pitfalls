# Pitfalls-Http

## 1. Request

### Request-Interception

#### Form-Submission

The interception of form submissions can be done in **two ways**, with the most common approach being the use of event listeners. By listening for the 'submit' event and then utilizing the `preventDefault` method, the default form submission behavior is halted. This allows for the execution of custom logic or validation before the actual submission takes place.

##### 1. Listener

```javascript
document.getElementById('form').addEventListener('submit', function(event) {
  event.preventDefault();
});
```

In my work, I encountered a situation where the form is submitted directly in a script, for example, using `form.submit()`. By manipulating the form submission through scripts, it bypasses my interception, and I cannot prevent the request. In such cases, we can employ the second method.



##### 2. HTMLFormElement

We can intercept by modifying the prototype chain of `HTMLFormElement`. In this case, when we call the `submit()` method and trigger the submit event, the interception logic will take effect.

```javascript
var originalSubmit = HTMLFormElement.prototype.submit;

HTMLFormElement.prototype.submit = function() {
    var action = this.getAttribute('action');
    var method = this.getAttribute('method');

    console.log('Form submission intercepted!');
    
    if (method && method.toLowerCase() === 'post') {
        console.log('POST request to:', action);
    } else {
        console.log('Non-POST request. Method:', method, ' Action:', action);
    }

    originalSubmit.call(this);
};
```



**Considerations:**

1. Listeners are used to intercept button clicks.
2. Modifying the prototype chain of `HTMLFormElement` is employed for intercepting the `submit()` event called by scripts.



## 2. API

### API Design

Designing a typical API request typically involves considerations in the following aspects:

1. HTTP Method: Defines the type of operation for the request. Common HTTP methods include GET, POST, PUT, DELETE, etc. GET is used to retrieve resources, POST to create resources, PUT to update resources, and DELETE to delete resources.
2. URL (Uniform Resource Locator): Contains the target address of the request. The URL should clearly specify the API endpoint and possible query parameters. For example, https://api.example.com/users.
3. Headers: Contain metadata about the request, such as authentication information, content type, and request timestamp. For example, Content-Type: application/json indicates that the request body contains JSON-formatted data.
4. Request Body: Applicable to certain HTTP methods (e.g., POST and PUT). It contains the data to be sent to the server. The data format is typically specified by the Content-Type field in the headers and can be JSON, XML, form data, etc.
5. Authentication: If the API requires authentication, the request needs to include appropriate authentication information, such as an API key or token. Authentication is usually passed through the headers, for example, Authorization: Bearer <token>.
6. Query Parameters: If the API supports query parameters, they can be added to the URL to filter, sort, or limit results. For example, https://api.example.com/users?role=admin&active=true.
7. Path Parameters: Sometimes, the URL of the API request includes path parameters used to specify a specific part of the resource. For example, https://api.example.com/users/{id}.
8. Error Handling: When designing requests, consider potential errors and establish suitable error-handling mechanisms. Error messages are typically returned in the form of standard HTTP status codes and error messages.

#### API Headers

The response headers of an API can contain various information, and the specific content depends on the requirements and implementation of your API. Here are some common response header fields:

**Content-Type:** Specifies the media type of the response body. Common values include application/json, text/html, application/xml, etc.

```http
Content-Type: application/json
```

**Content-Length:** Indicates the length of the response body in bytes.

```http
Content-Length: 1024
```

**Cache-Control:** Controls caching behavior, including the maximum cache duration and whether it can be cached.

```http
Cache-Control: max-age=3600, public
```

**Date:** Indicates the date and time when the response was sent.

```http
Date: Tue, 21 Feb 2024 12:34:56 GMT
```

**Server:** Describes information about the server.

```http
Server: My Awesome Server
```

**ETag:** Used to verify if the resource in the cache is the same as on the server.

```http
ETag: "abc123"
```

**Location:** In a redirection response, indicates the URL to which the client should redirect.

```http
Location: https://example.com/new-location
```

**Access-Control-Allow-Origin:** Allows sources for cross-origin requests.

```http
Access-Control-Allow-Origin: *
```

**Content-Disposition:** Specifies how to display the response result, especially for file downloads.

```http
Content-Disposition: attachment; filename="example.txt"
```

**Set-Cookie:** Sets a cookie in the response.

```http
Set-Cookie: user_id=123; Max-Age=3600; Path=/
```

These are just some common response header fields, and their specific usage depends on the design and requirements of your API. When designing an API, ensure that the response headers include appropriate information for clients to parse and handle the response correctly.



##### 1. Content-Type

In addition to application/json, the Content-Type in the HTTP header can be set to other common media types depending on the requirements of your application and API. Here are some common Content-Type types:

**Text Types:**

- **text/plain:** Plain text without any formatting.
- **text/html:** HTML documents.

**XML Types:**

- **application/xml:** Data in XML format.

**Form Types:**

- **application/x-www-form-urlencoded:** Form data, typically used for HTML form submissions.
- **multipart/form-data:** Multi-part form data, commonly used for file uploads.

**JSON Type:**

- **application/json:** Data in JSON format.

**Binary Type:**

- **application/octet-stream:** Binary data, typically used for file downloads.

**JavaScript Type:**

- **application/javascript:** JavaScript scripts.

**CSS Type:**

- **text/css:** CSS style sheets.

**Image Types:**

- **image/jpeg, image/png, image/gif:** Different formats of images.

When setting the Content-Type, ensure to choose the appropriate type to ensure the client can correctly parse the response. Typically, use application/json for returning JSON data, text/plain or text/html for text data, and multipart/form-data for file uploads, among others.



##### 2. Content-Length

`Content-Length` is an HTTP header field used to indicate the length, in bytes, of the HTTP message body being sent to the server or returned from the server. This field is typically employed to determine the end position of the message body, ensuring proper parsing of the HTTP message.

In HTTP requests, when a message body is included (for example, in a POST request), the client uses `Content-Length` to inform the server about the size of the message body. The server can then utilize this information to properly read and parse the request's message body.

In HTTP responses, the server uses the `Content-Length` header to indicate the size of the response message body being sent to the client. The client can use this information to ensure the complete reception of the response message body.



##### 3. Cache-Control

`Cache-Control` is an HTTP header field used to control caching behavior. This field provides instructions on how caching should store, retrieve, and revalidate resources. In HTTP requests and responses, `Cache-Control` offers flexible control over caching behavior, aiding in performance optimization and reducing network traffic.

Here are some common `Cache-Control` directives:

1. **public/private**: Specifies whether the response can be stored by any cache (public) or is limited to a single user's private cache (private).

2. **max-age**: Specifies the maximum time, in seconds, from the request time that the resource should be considered fresh. Beyond this time, the cache is deemed expired and requires revalidation.

3. **no-cache**: Indicates that the cache should not use the response directly, and a request needs to be sent to the origin server to obtain the complete response. However, it still allows intermediate caches (proxies) to validate whether the resource is still valid.

4. **no-store**: Indicates that neither the request nor the response should be stored, disallowing any caching.

5. **must-revalidate**: Instructs caches to revalidate expired resources before using them. If a resource is expired and cannot be validated, the cache must fetch the resource again.

These are just some common directives of the `Cache-Control` header; there are additional directives and parameters that can be combined as needed. The flexibility of this header allows developers to precisely control caching behavior when balancing performance and real-time requirements.



##### 4. Date



##### 5. Server

The `Server` response header is used to specify information about the web server software currently in use. It allows the client to understand the type and version of the server sending the response.

For example, a `Server` header might contain information like this:

```
Server: Apache/2.4.41 (Unix) OpenSSL/1.1.1c
```

This tells the client that the server is using Apache software, version 2.4.41, running on a Unix operating system, and using OpenSSL encryption library version 1.1.1c.

As a developer, using the `Server` header can serve the following purposes:

1. **Server Identification:** Provide client-side information identifying the current server. This is useful for debugging and monitoring network traffic.

2. **Security and Compliance:** Ensure the use of the latest and secure server software versions. Some security and compliance standards may require exposing the server software's version information to ensure timely security updates.

3. **Debugging and Maintenance:** During development and maintenance, understanding the server's version and configuration can assist in resolving issues specific to the server software.

Note that sometimes, to enhance security or for other security policy reasons, some websites and applications may choose to hide or modify the `Server` header to prevent the leakage of too much information.



##### 6. **ETag**

`ETag` (Entity Tag) is a part of the HTTP response header, used to identify the version of a resource. It is typically generated by the server and aids in cache management between the client and server to determine if the resource has been modified.

**Purpose:**

1. **Cache Validation:** When a client requests a specific resource, it can include the `ETag` value received during the last resource retrieval. The server can use this value to determine if the client's cached resource is still the latest. If the resource has changed, the server returns the new resource; if unchanged, the server can respond with a `304 Not Modified` status, instructing the client to continue using the cached version.

2. **Reduce Bandwidth Consumption:** Using `ETag` helps minimize unnecessary bandwidth consumption. If the resource hasn't changed, the server only needs to return a `304 Not Modified` response instead of transmitting the entire resource content.

`ETag` is a crucial mechanism in HTTP caching and validation, contributing to optimizing network performance and reducing unnecessary resource transfers.



##### 7. **Location**

The `Location` response header determines the new location (URL) to which the client should be redirected. When the server returns a response that includes the `Location` header, it informs the client to move the target of the current request to the specified new URL.

**Usage:**

1. **Redirection Responses:** Its primary purpose is when the server wants the client to redirect the current request to another URL. This could be due to resource movement, permanent redirection, temporary service unavailability, etc.

2. **POST-Redirect-GET Pattern:** In web development, POST requests are commonly used to submit form data. After receiving these POST requests, the server may return a 3xx redirection response, including the new URL in the `Location` header. This is done to prevent users from resubmitting the same form data when refreshing the page, implementing the POST-Redirect-GET pattern.

**Example:**

```http
HTTP/1.1 302 Found
Location: https://example.com/new-location
```

In this example, the server returns a status code of `302 Found` and specifies the new URL in the `Location` header. After receiving this response, the client automatically initiates a request to the new URL, achieving redirection.

**Considerations:**

- **Choice of Status Code:** Common status codes include `302 Found` (temporary redirection) and `301 Moved Permanently` (permanent redirection). The choice of status code depends on the nature of the redirection.

- **Absolute and Relative Paths:** `Location` can contain absolute or relative paths. If it is a relative path, it will be resolved based on the current request's URL.

- **URL Encoding:** If the URL contains special characters or non-ASCII characters, it should be URL-encoded.

In summary, the `Location` header is a core mechanism for implementing redirection, allowing the server to inform the client to move the target of the current request to a new URL.



##### 8. **Access-Control-Allow-Origin**

`Access-Control-Allow-Origin` is an HTTP header used to specify which domains (origins) have permission to access a resource. It is a key header in the CORS (Cross-Origin Resource Sharing) mechanism.

When a webpage attempts to make a cross-origin request using JavaScript, the browser first sends an OPTIONS preflight request to the server to determine whether the actual cross-origin request is allowed. The value of the `Access-Control-Allow-Origin` header in the OPTIONS preflight request and the response to the actual request is used to control access permissions.

**Meaning of Values:**

- **Specific Domain:** If set to a specific domain, it indicates that only webpages from that domain are allowed to access the resource. For example, `Access-Control-Allow-Origin: https://example.com` allows requests from `https://example.com`.

- **Wildcard `*`:** If set to `*`, it means any webpage from any domain is allowed to access the resource. For example, `Access-Control-Allow-Origin: *` allows requests from any domain.

**Examples:**

```http
Access-Control-Allow-Origin: https://www.google.com
```

This example indicates that only requests from `https://example.com` are allowed to access the resource.

```http
Access-Control-Allow-Origin: *
```

This example indicates that requests from any domain are allowed to access the resource.

In practical applications, it is important to choose an appropriate value for `Access-Control-Allow-Origin` based on specific security requirements and business logic. Typically, it is advisable to only allow necessary domains to access resources to enhance security.



> Please note the difference between `Access-Control-Max-Age` and `Cache-Control`.

**Access-Control-Max-Age:**

Purpose: Primarily used for Cross-Origin Resource Sharing (CORS). When the browser initiates a preflight request (OPTIONS request) to the server through a cross-origin request, the server can use the `Access-Control-Max-Age` header to inform the browser that, for the same cross-origin request, it doesn't need to send another preflight request within the specified time (in seconds). It can directly use the previously obtained authorization result.

Example:

```javascript
res.setHeader('Access-Control-Max-Age', '3600');
```

**Cache-Control:**

Purpose: Used to control caching behavior. It guides the browser on how long to store the response in local cache, whether to allow caching, whether revalidation is necessary, etc.

Examples:

```javascript
// Disable caching
res.setHeader('Cache-Control', 'no-store');

// Allow caching but validate with the server on every request
res.setHeader('Cache-Control', 'no-cache');

// Allow caching and specify a maximum caching time of one hour
res.setHeader('Cache-Control', 'public, max-age=3600');
```

In summary, `Access-Control-Max-Age` is mainly used for optimizing preflight requests in cross-origin requests, while `Cache-Control` is primarily used to control caching behavior. Although they serve different purposes, in practical applications, they may be used together to better control and optimize network requests and resource usage.



**Summary:**
In Cross-Origin Resource Sharing (CORS), `Access-Control-Allow` is a common prefix followed by different header information to control permissions for cross-origin requests. Here are some common `Access-Control-Allow` headers:

###### 1. Access-Control-Allow-Origin:

- Controls which domains are allowed to access the resource. It can be set to a specific domain, multiple domains, or use the wildcard `*` to allow all domains.

  ```http
  Access-Control-Allow-Origin: https://example.com
  ```

###### 2. Access-Control-Allow-Methods:

- Specifies the allowed HTTP methods. Typically set to the methods supported by the request handler.

  ```http
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  ```

###### 3. Access-Control-Allow-Headers:

- Specifies the allowed HTTP headers. Before sending the actual request, the browser sends a preflight request (OPTIONS request), and this header specifies additional headers allowed in the preflight request.

  ```http
  Access-Control-Allow-Headers: Content-Type, Authorization
  ```

###### 4.  Access-Control-Allow-Credentials:

- Indicates whether credentials (such as cookies, HTTP authentication) are allowed to be carried in the request.

  ```http
  Access-Control-Allow-Credentials: true
  ```

###### 5. Access-Control-Expose-Headers:

- Specifies which response headers can be exposed to front-end JavaScript code.

  ```http
  Access-Control-Expose-Headers: Content-Length, ETag
  ```

###### 6. Access-Control-Max-Age:

- Specifies the validity period of the preflight request, i.e., how long before it needs to be sent again.

  ```http
  Access-Control-Max-Age: 3600
  ```

These headers can be used individually or together to meet specific cross-origin requirements. Depending on the situation, you may need to set some or all of them.



##### 9. **Content-Disposition**

`Content-Disposition` is an HTTP header used to indicate how the response content should be handled and whether there should be a prompt for the user, commonly used in file download scenarios.

This header has two main purposes:

1. **File Download Prompt:**
   - When the server response includes a file for download, setting the `Content-Disposition` header can prompt the browser to display a file download dialog instead of opening the file directly in the browser.

     ```http
     Content-Disposition: attachment; filename="example.txt"
     ```

   - In the above example, `attachment` indicates that the file should be downloaded, and `filename="example.txt"` specifies the name of the downloaded file.

2. **Inline Embedding:**
   - If you want the browser to display the content directly rather than initiating a download, you can use `inline`.

     ```http
     Content-Disposition: inline
     ```

   - This is useful for content like PDF files that can be displayed directly in the browser.

In summary, `Content-Disposition` is a versatile header that allows the server to instruct the browser on how to handle the response content.



##### 10. **Set-Cookie**

`Set-Cookie` is a field set by the server in the HTTP response header to store cookies on the client. It is commonly used to store session information or identify user-related details on the client side.

When the server includes the `Set-Cookie` field in the response header, the browser stores the cookie and sends it to the server in subsequent requests. The storage and sending of cookies can be subject to certain security policies implemented by the browser, such as the SameSite attribute.

In your specific context, if the server sets `Set-Cookie` in the response, the browser will store the cookie upon receiving that response. In subsequent requests, the browser attaches the cookie to the request header and sends it back to the server.

**Note:** The specific impact of `Set-Cookie` also depends on the browser's same-origin policy and the cookie's attribute settings. Ensure that your server-side and client-side code handle cookies correctly.



`httpOnly`, `secure`, and `SameSite` are properties related to HTTP cookies. Their functions are as follows:

1. **httpOnly:**
   - **Function:** After setting `httpOnly`, JavaScript cannot access the cookie through `document.cookie`, providing additional protection against Cross-Site Scripting (XSS) attacks.
   - **Setting:** When setting a cookie on the server side, add the `HttpOnly` attribute.

   ```javascript
   res.setHeader('Set-Cookie', 'myCookie=myValue; HttpOnly');
   ```

2. **secure:**
   - **Function:** When the `secure` attribute is set, the cookie is only sent to the server when using a secure connection (HTTPS). This helps prevent the transmission of sensitive information over non-secure connections.
   - **Setting:** When setting a cookie on the server side, add the `Secure` attribute.

   ```javascript
   res.setHeader('Set-Cookie', 'myCookie=myValue; Secure');
   ```

3. **SameSite:**
   - **Function:** The `SameSite` attribute controls whether a cookie should be sent in cross-site requests. It has three possible values: `Strict`, `Lax`, and `None`.
      - `Strict`: Send the cookie only if the request URL matches the cookie. It does not allow any third-party requests to send the cookie.
      - `Lax`: Allows the cookie to be sent only with top-level navigations (e.g., clicking on a link). It also sends the cookie when submitting a form via the POST method.
      - `None`: Always send the cookie, even with cross-site requests.
   - **Setting:** When setting a cookie on the server side, add the `SameSite` attribute.

   ```javascript
   res.setHeader('Set-Cookie', 'myCookie=myValue; SameSite=Strict');
   ```

   ```javascript
   res.setHeader('Set-Cookie', 'myCookie=myValue; SameSite=Lax');
   ```

   ```javascript
   res.setHeader('Set-Cookie', 'myCookie=myValue; SameSite=None; Secure');
   ```

When setting these properties, choose based on specific requirements and security considerations. For example, when you want a cookie to be transmitted only over a secure connection and prevent JavaScript access, you can use `httpOnly`, `secure`, and `SameSite=None` attributes together.



## 3. Ways to Solve Cross Origin 

### 1. CORS(Cross Origin Resource Sharing)

**Definition:**

CORS is a W3C standard. It allows explorer to send XMLHttpRequest to cross-origin web server, hence overcome the restriction that AJAX can only be used within same origin. The use of CORS requires support for both client and server.

**Features:**

- Auto completion by explorer, user participation not needed
- No difference between CORS and same-origin AJAX communication
- Key part is server(as long as the server implements CORS interface, corss-origin communication can be achieved)



### 2. JSONP(JSON with Padding)

**Principle:** calling js files on the web is not influenced by same-origin policy of the explorer, hence cross origin communication can be achieve via <script>.

```js
function callback(res){
    console.log(res);
}

let script = document.createElement('script');

script.src = 'https://www.example.com/login?username=admin&callback=callback';

document.body.appendChild(script);
```

**Advantages:**

- Not restricted by same-origin policy, unlike the way XMLHttpRequest object achieving Ajax request.
- Has great compatibility and can be run in even very old explorers.
- No XMLHttpRequest or ActiveX support needed and can get the response via calling callback functions after the completion of request.



**Disadvantages:**

- Only support `GET` method and no way for other HTTP request methods.
- Only support the case for cross origin HTTP request and cannot solve the data communication problem of webpages from two different domains or between iframes.



**Example:**

```js
function makeJSONPRequest(url, callbackName) {
    var script = document.createElement('script');

    script.src = url + '?callback=' + callbackName;

    document.body.appendChild(script);
}

function handleJSONPResponse(response) {
    console.log(response);
}

var apiUrl = 'https://www.baidu.com/my/favorites?page=1';
var callbackFunction = 'handleJSONPResponse';

makeJSONPRequest(apiUrl, callbackFunction);
```

**Note:** If you use an HTTP request instead of HTTPS, it will fail. The reason is that your page is loaded via HTTPS, but the JSONP request's URL is using HTTP, which violates the browser's security policy. Browsers do not allow loading insecure (HTTP) resources on HTTPS pages, leading to a Mixed Content error.



### 3. Nginx Proxy

Nginx (pronounced "engine-x") is an open-source, high-performance, lightweight web server that can also be used as a reverse proxy server, load balancer, and HTTP cache, among other functionalities. Typically, Nginx is employed to handle static files of web applications, proxy requests to application servers, and perform various other network services.

Pure frontend development usually does not involve server-side work, so in general, frontend developers do not need to set up an Nginx server. Nginx is more commonly used in backend development and server configuration to provide web services or act as a reverse proxy. Frontend engineers typically host their static web pages or frontend applications on servers managed by backend teams or specialized hosting service providers.

If you simply want to test or learn Nginx locally, you can install and run the Nginx server, configuring it to serve static files. However, in real production environments, Nginx is often used in conjunction with backend services to build a complete web application architecture.

Principle: setting up a proxy server(domain name is the same as Domian1) via Engine X as a jump server, reverse proxy to access Domain2 interface; and can modify domain information inside the cookie at the same time, making it easy for writing in the cookie, hence achieving cross-origin communication.

**Example1:**

```nginx
server{
    listen: 80;
    server_name www.domain1.com;
    location / {
        proxy_pass http://www.domain2.com:8080/;
    }
}
```

**Example2:**

```nginx
server{
    listen: 81;
    server_name www.domain1.com;
    location / {
        proxy_pass http://www.domain2.com:8081/;
    }
}
```

**Example3:**

```nginx
server{
    listen: 80;
    server_name www.domain1.com;
    location / {
    	// reverse proxy
        proxy_pass http://www.domain2.com:8080/;

    	// cookie
    	proxy_pass_domain www.domain2.com www.domain1.com;

    	index index.html index.html
    
    	add_header Access-Control-Allow-Origin http://www.domain1.com;
    
    	add_header Access-Control-Allow-Credentials true;
    }
}
```



### 4. Node Proxy

You can refer to the chapter on **Reverse Proxy** in the article **Pitfalls-Mock**.

