# Pitfalls-Mock

## Building a simple mock from scratch

### Building package.json

#### 1. Install Dependencies


1. **axios**: Used to simulate HTTP requests in the Mock Server. It is employed to initiate simulated HTTP requests, simulating communication between the client and the server.

2. **body-parser**: Used to parse data from the request body. In the Mock Server, it helps parse data in various formats (JSON, forms) from POST requests.

3. **btoa**: Provides Base64 encoding functionality. In the Mock Server, it might be used to encode or decode headers in Base64, simulating certain security mechanisms.

4. **commander**: Used to build command-line tools. In the Mock Server, it could be used to create command-line scripts for easy initiation, configuration, or management of the Mock server.

5. **connect-busboy**: Handles file uploads. In the Mock Server, it could be used to simulate file uploads by handling files from incoming requests.

6. **cookie-parser**: Parses cookies from requests. In the Mock Server, it assists in simulating the handling of cookies.

7. **cors**: Manages Cross-Origin Resource Sharing. In the Mock Server, cors might be used to simulate cross-origin requests, allowing or denying access from specific domains.

8. **cross-env**: Sets environment variables. In the Mock Server, it might be used to configure different Mock behaviors in different environments.

9. **express**: Builds a web server. In the Mock Server, express can be used to create a simple server that handles HTTP requests and returns simulated responses.

10. **express-http-proxy**: Used for proxying HTTP requests. In the Mock Server, it can be used to proxy certain requests to other servers, simulating real backend services.

11. **express-session**: Manages session information. In the Mock Server, it might be used to simulate session management, handling user login states, etc.

12. **express-urlrewrite**: Handles URL rewriting. In the Mock Server, it can be used to rewrite the URLs of requests to match the simulated routing rules.

13. **glob**: Matches file paths. In the Mock Server, it might be used to find specific files or directories for loading Mock data.

14. **http-proxy-middleware**: Used for proxying HTTP requests. Similar to express-http-proxy, it can be used in the Mock Server to proxy requests to other servers.

15. **JSONPath**: Used for extracting specific fields from JSON data. In the Mock Server, it might be used to process JSON data in requests or responses.

16. **lodash**: Provides various utility functions for JavaScript. In the Mock Server, it can be used for data manipulation, making Mock data generation more flexible and convenient.

17. **node-fetch**: Used for making HTTP requests in Node.js. In the Mock Server, it can replace axios for simulating server-side requests to other services.

18. **path-to-regexp**: Handles URL path matching. In the Mock Server, it might be used to define routing rules, matching the paths of requests and generating corresponding Mock responses.

19. **request**: Used for making HTTP requests. Similar to axios, it can be used in the Mock Server to simulate client-side requests.



#### 2. Add Scripts

> Compared to Node, launching the application with `nodemon app.js` allows `Nodemon` to monitor all file changes and automatically restart `app.js` when needed.

```json
"scripts": {
	"start": "node app.js",
	"local": "nodemon app.js"
}
```

You also need install `nodemon`.



### Building app.js

#### 1. Importing

```js
const _ = require('lodash');
const axios = require('axios');
const bodyParser = require('body-parser');
const connectBusboy = require('connect-busboy');
const cors = require('cors');
const express = require('express');
const fetch = require('node-fetch');
const fs = require('fs');
const glob = require('glob');
const http = require('http');
const https = require('http');
const path = require('path');
const proxy = require('express-http-proxy');
const request = require('request');
const urlRewrite = require('express-urlrewrite');
```



#### 2. Creating an instance

1. Calling the express() function creates an Express application.
2. This application can be considered as a `request handler` that can perform various operations (such as routing decisions, sending responses, etc.) upon receiving HTTP requests.
3. Assign the newly created Express application to the variable 'app'. This way, the 'app' variable can be used to configure the application (through middleware), add route handlers, and more.

```js
const app = express();
```



#### 3. Define routes

Defined a simple route.

```js
app.post('/M/simple', (req, res) => {
    res.send('Mock Simple Route Test');
});
```

You also can define a complex route. It will be mentioned later.

```js
const router = express.Router();
```



#### 4. Listening on port

**Considerations:** '0.0.0.0' is the IP address to which the server is bound. When you set it to '0.0.0.0', it means the server will accept requests from any IP address. This makes your server publicly accessible, not just locally. If you only want local access, you can set it to 'localhost' or '127.0.0.1'."

```js
const port = 4500;
http.createServer(app).listen(port, '0.0.0.0', () => {
    console.log(`Server start on port ${port}`);
});
```



## Enhancing the mock

### Resolving Cross-Origin

If you open the browser normally instead of in cross-origin mode, and enter the following code in the console, you will encounter an error like this.

```js
fetch('http://localhost:4500/M/simple', {
    method: "POST"
})
```

**Warning:** Access to fetch at 'http://localhost:4500' from origin 'null' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

CORS (Cross-Origin Resource Sharing) is a security mechanism used to control how web pages loaded from one domain can request resources from another domain. The error you see in the browser indicates that your request involves cross-origin interactions, and the server is not properly configured for CORS.

Now we need to resolve this issue.



#### CORS middleware

We just need to add below code:

```js
// 使用 cors 中间件处理CORS头
app.use(cors());
```

Now if you send the request again, the problem will be solved.



### Define complex routing

`router` is an Express router object created by calling `express.Router()`. `router.route` allows you to define routes for multiple HTTP methods on the same path without specifying the path multiple times. This enables you to chain multiple methods and define multiple handlers for the same path. This approach is more suitable for large applications or scenarios where multiple methods need to be defined for the same path.

1.  Create router

```js
const router = express.Router();
```

2. Define routes

```js
router.route('/M/complex').post((req, res) => {
    res.send('Test Mock Complex Route Post');
});
```

You can also define both methods together using two approaches:

```js
router.route('/M/complex').get((req, res) => {
    res.send('Test Mock Complex Route Get');
}).post((req, res) => {
    res.send('Test Mock Complex Route Post');
});
```

Most of the time, it can be used to return JSON files:

```js
const pocJSON = require('./poc.json');
pocRouter.route('/M/poc').get((req, res) => {
    res.json({
        'method': 'get'
    });
}).post((req, res) => {
    res.json(pocJSON);
});
```



3. Use the router which defined

```js
app.use('', router);
```



4. Adjust project structure

You can export the router as a module to make app.js look cleaner and also to organize the functionality of your configured routes more clearly.

```js
// poc.js
const express = require('express');
const pocRouter = express.Router();

pocRouter.route('/M/complex').get((req, res) => {
    res.send('Test Mock Complex Route Get');
}).post((req, res) => {
    res.send('Test Mock Complex Route Post');
});

const pocJSON = require('./poc.json');
pocRouter.route('/M/poc').get((req, res) => {
    res.json({
        'method': 'get'
    });
}).post((req, res) => {
    res.json(pocJSON);
});

module.exports = pocRouter;

// app.js
const pocRouter = require('./--poc/poc');
app.use('', pocRouter);
```



### Add response headers

Attributes and their functions of the header can be referred to in another article "Pitfalls-Http."

This article is just a simple tutorial.

#### 1. Content-Type

You can use different **Content-Type** to return various values. For more details, you can check the network tab in your browser.

```js
router.route('/api/example')
    .get((req, res) => {
        const responseData = { message: 'GET request handled' };
        res.setHeader('Content-Type', 'text/plain');
        res.status(200).json(responseData);
    })
    .post((req, res) => {
        const responseData = { message: 'POST request handled' };
        res.setHeader('Content-Type', 'application/json');
        res.status(200).json(responseData);
    });
```

```js
fetch('http://localhost:4500/api/example', {
    method: "GET",
    headers: {
        "Content-Type": "application/json"
    }
}).then(response => {
    console.log(response)
})
```

**Considerations:** 

1. But we found that during the process of sending an API from the frontend, it still works even if the Content-Type used is different from what the backend returns. 

2. At the same time, when sending the API with a Content-Type, the browser initiates a preflight check.

**Reply:**

1. Your code indeed sets different Content-Types, but in this scenario, Content-Type is primarily used to inform the server about the type of the request body. In your GET request, Content-Type is set to "application/json," but typically, GET requests do not include a request body. Hence, servers usually ignore it.

2. When browsers send cross-origin requests, especially those with special headers (such as custom headers) or using certain types of request methods (e.g., PUT, DELETE), it triggers a "preflight" request for cross-origin requests. This is because, in a cross-origin situation, the browser needs to ensure whether the server allows the actual request. The preflight request uses the OPTIONS method.

   The purpose of the preflight request is to ensure that the server supports cross-origin requests and to check if there are any restrictions on communication between the browser and the server before sending the actual request. During this preflight request, the browser checks for CORS-related headers (such as Access-Control-Allow-Origin, Access-Control-Allow-Methods, etc.) and whether specific request headers are allowed.

   Setting the Content-Type in the request header is not the sole condition triggering a preflight request; other scenarios might also lead to preflight requests. For instance, using non-simple request methods (other than GET, HEAD, POST) or non-simple request header fields (such as custom header fields) could also trigger preflight requests.

   Of course, if you ensure that the Content-Type in the request header is for a simple request (like application/x-www-form-urlencoded, multipart/form-data, text/plain), you're using a simple request method (GET, HEAD, POST), and there are no custom request header fields, then the browser won't send a preflight request and will directly send the actual request.

   In summary, the CORS (Cross-Origin Resource Sharing) mechanism is a security strategy that ensures cross-origin requests won't be triggered without explicit authorization. The preflight request is a part of CORS, and by inspecting the server's response headers, the browser can decide whether to proceed with the actual request.



#### 2. Content-Length

**Set Length**

```js
router.route('/api/example')
    .get((req, res) => {
        const responseData = { message: 'GET request handled' };
        res.setHeader('Content-Type', 'text/plain');
        res.status(200).json(responseData);
    })
    .post((req, res) => {
        const responseData = { message: 'POST request handled' };
        const responseBody = JSON.stringify(responseData);
        res.setHeader('Content-Type', 'application/json');
        res.setHeader('Content-Length', Buffer.byteLength(responseBody));
        res.status(200).json(responseData);
    });
```



#### 3. Cache-Control

`res.setHeader('X-Custom-Header', 'Custom-Value');` This line of code sets a custom HTTP response header. Let's break down some key parts of it:

- `res.setHeader`: This is the method in the Express framework used to set HTTP response headers.

- `'X-Custom-Header'`: This is a custom response header name. Response headers starting with 'X-' are typically used for custom headers specific to an application or service.

- `'Custom-Value'`: This is the value you set for the custom response header `'X-Custom-Header'`. In practice, you can set it to any string value you find appropriate to convey custom information in the response.

In this example, by setting `'X-Custom-Header'`, you add a custom identifier or information to the response. This can be useful for identifying the processing status of a request, indicating the server version, or conveying other information relevant to the application. Browsers or clients can read this response header to obtain additional information or perform specific logic.

```js
router.use((req, res, next) => {
    res.setHeader('X-Custom-Header', 'Custom-Value');
	
    if (req.method === 'GET') {
        res.setHeader('Cache-Control', 'public, max-age=3600');
    } else {
        res.setHeader('Cache-Control', 'no-store');
    }

    next();
});
```

**Considerations:** 
In Express, `next` is a callback function used to pass control to the next middleware function in the middleware stack. Each middleware function takes three parameters: `req` (request object), `res` (response object), and `next` (callback function).

When you call `next()`, Express moves to the next middleware that matches the request. If there are no more middleware functions, Express will end the request-response cycle. In this case, the purpose of `next` is to instruct Express to continue processing the next step in the request.

Here, the purpose of `next()` is to pass the request to the route handler or the next middleware. If there are other routes or middleware defined after `router.use`, they will have a chance to handle the request. If there are no other matching middleware, the request will proceed to the route handler.



#### 4. Date

```js
router.use((req, res, next) => {
    res.setHeader('X-Custom-Header', 'Custom-Value');
    res.setHeader('Date', new Date().toUTCString());

    if (req.method === 'GET') {
        res.setHeader('Cache-Control', 'public, max-age=3600');
    } else {
        res.setHeader('Cache-Control', 'no-store');
    }

    next();
});
```

**Considerations:** 

In general, the `Date` header is **`automatically`** set by the server to indicate the date and time the response was sent. You don't need to explicitly set the `Date` header; Express automatically adds it to each response.

In your code, setting the `Date` header is optional because Express defaults to setting this header. In fact, if you don't manually set the `Date` header, Express will automatically add the current date and time to the response.

Therefore, if you don't explicitly set the `Date` header, the browser will still receive the `Date` header with the value representing the date and time when the server sent the response. You can check the response headers using the network tab in the browser developer tools to confirm if the `Date` header is correctly set.

If you want to ensure that the `Date` header accurately reflects the time the server sent the response, you can use code like `res.setHeader('Date', new Date().toUTCString());`. However, please note that this may have limited impact on browser behavior as browsers typically handle the `Date` header on their own.



#### 5. Server

Add:

```js
res.setHeader('Server', "VL's Server");
```



#### 6. **ETag**

Install require package `crypto` at first

- createHash('md5'): Creates a hash object using the MD5 algorithm.
- update(content): Adds the data to be hashed to the hash object.
- digest('hex'): Retrieves the hash value in hexadecimal format.

```js
const content = "VL's ETAG";
const etag = crypto.createHash('md5').update(content).digest('hex');
const ifNoneMatch = req.headers['if-none-match'];

if (ifNoneMatch === etag) {
    res.status(304).end();
} else {
    res.setHeader('ETag', etag);

    const responseData = { message: 'GET request handled' };
    res.status(200).json(responseData);
}
```

```js
fetch('http://localhost:4500/api/example', {
    method: "GET",
    headers: {
        "Content-Type":a "application/json",
        "If-None-Match": "be1f50fbe9e9ba6774d634d052116e68"
    }
}).then(response => {
    if (response.status === 302) {
        const redirectUrl = response.headers.get('Location');
        console.log('Redirecting to:', redirectUrl);
        window.location.href = redirectUrl;
    } else if (response.status === 304) {
        console.log('Resource not modified, using cached version.');
    } else {
        const etag = response.headers.get('ETag');
        console.log('ETag:', etag);
        return response.json();
    }
}).then(data=>{
    console.log(data);
}).catch(error => {
    console.error('Fetch error:', error);
});
```

However, `response.headers` is an empty object. We will address how to resolve such issues later on.



#### 7. **Location**

The purpose of `encodeURIComponent` is to convert special characters in a string into their URI-encoded forms, ensuring they can be safely used as part of a URI. Special characters include those that have special meanings in a URI, such as question mark (?), equal sign (=), ampersand (&), and others.

**Usage:**

```js
const redirectTarget = "https://www.baidu.com";

const apiUrl = `http://localhost:4500/api/example?redirect=${encodeURIComponent(redirectTarget)}`;

fetch(apiUrl, {
    method: "GET",
    headers: {
        "Content-Type": "application/json",
        "If-None-Match": "be1f50fbe9e9ba6774d634d052116e68"
    }
}).then(response => {
    if (response.status === 302) {
        const redirectUrl = response.headers.get('Location');
        console.log('Redirecting to:', redirectUrl);
        window.location.href = redirectUrl;
    } else if (response.status === 304) {
        console.log('Resource not modified, using cached version.');
    } else {
        const etag = response.headers.get('ETag');
        console.log('ETag:', etag);
        return response.json();
    }
}).then(data => {
    if (data !== null) {
        console.log(data);
    }
}).catch(error => {
    console.error('Fetch error:', error);
});
```

Due to some issues, we are unable to retrieve the header. Don't worries, we will resolve it later.



#### 8.  **Access-Control-Allow-Origin**

**What is `cors()` do?**

1. **Setting the response header `Access-Control-Allow-Origin`：** is the most crucial part of CORS. It informs the browser which domains are allowed as the origin for cross-origin requests. Typically, it can be set to a specific domain or * to allow requests from any domain.
2. **Handling preflight requests (OPTIONS requests):** For certain requests (such as those with custom request headers or using non-simple methods), the browser sends an OPTIONS preflight request to determine if it is safe to send the actual request. The `cors()` middleware automatically handles this preflight request, ensuring the correct response headers, including `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and `Access-Control-Max-Age`.
3. **Allowing credentials to be carried:** By setting the `credentials` option, you can allow the browser to carry identity credentials (such as cookies, HTTP authentication, etc.) in cross-origin requests.
4. **Other related CORS headers:** For example, `Access-Control-Expose-Headers` can specify which response headers can be exposed to front-end JavaScript, and `Access-Control-Allow-Credentials` indicates whether the request is allowed to carry identity credentials.

> When sending a request to https://www.google.com, is there any issue with the following code?

```js
// app.use(cors());

// --------------------Routes-------------------- //
router.use((req, res, next) => {
    // res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Origin', 'https://www.google.com');
    res.setHeader('X-Custom-Header', 'Custom-Value');
    res.setHeader('Date', new Date().toUTCString());
    res.setHeader('Server', "VL's Server");
    
    if (req.method === 'GET') {
        res.setHeader('Cache-Control', 'public, max-age=3600');
    } else {
        res.setHeader('Cache-Control', 'no-store');
    }

    next();
});
```

The request fails due to a preflight OPTIONS request, and since Access-Control-Allow-Headers is not set on the backend, the request is unsuccessful.

So, you also need to add the following line of code to make request `success`:

```js
// res.setHeader('Access-Control-Allow-Headers', '*');
res.setHeader('Access-Control-Allow-Headers', 'Content-Type, If-None-Match, Referer, Sec-Ch-Ua, Sec-Ch-Ua-Mobile, Sec-Ch-Ua-Platform, User-Agent');
```

Now, let's add `Access-Control-Max-Age` and `Access-Control-Allow-Methods`.



**Complete example:**

```js
router.use((req, res, next) => {
    res.setHeader('Access-Control-Allow-Origin', '*');
    // res.setHeader('Access-Control-Allow-Origin', 'https://www.google.com');
    res.setHeader('Access-Control-Allow-Headers', '*');
    // res.setHeader('Access-Control-Allow-Headers', 'Content-Type, If-None-Match, Referer, Sec-Ch-Ua, Sec-Ch-Ua-Mobile, Sec-Ch-Ua-Platform, User-Agent');
    res.setHeader('Access-Control-Allow-Methods', '*');
    res.setHeader('Access-Control-Max-Age', '3600');

    res.setHeader('X-Custom-Header', 'Custom-Value');
    res.setHeader('Date', new Date().toUTCString());
    res.setHeader('Server', "VL's Server");

    if (req.method === 'GET') {
        res.setHeader('Cache-Control', 'public, max-age=3600');
    } else {
        res.setHeader('Cache-Control', 'no-store');
    }

    next();
});
```

You can observe the time spent in preflight by checking `Access-Control-Max-Age` to determine whether it was successful.



#### 9. **Content-Disposition**

Code:

```js
 res.setHeader('Content-Disposition', 'attachment; filename="example.txt"');
```



#### 10. **Set-Cookie**

**CORS MiddleWare Code:**

```js
const corsOptions = {
    origin: 'https://www.google.com',
    credentials: true,
};
app.use(cors(corsOptions));
```

**Node:**
When dealing with cross-origin requests, browsers enforce a security mechanism known as the "Same-Origin Policy," which prevents pages from making HTTP requests to a different domain. Cross-Origin Resource Sharing (CORS) is a mechanism that allows authorized access to resources by adding HTTP headers, specifying which origins are permitted.

In `corsOptions`, `origin: 'https://www.google.com'` indicates that requests are only allowed from the origin `https://www.google.com`. `credentials: true` means allowing requests that include credentials (such as cookies, HTTP authentication).

By using `app.use(cors(corsOptions))`, these CORS options are applied to the entire Express application, ensuring proper configuration of headers when handling cross-origin requests. This allows specific origins to access resources and handles credentials when necessary.

This is crucial for securely handling cross-origin requests and provides the server with more refined control over such requests.



**Fetch Function:**

```js
const redirectTarget = "https://www.baidu.com/";

const apiUrl = `http://localhost:4500/api/example?redirect=${encodeURIComponent(redirectTarget)}`;

fetch(apiUrl, {
    method: "GET",
    headers: {
        "Content-Type": "application/json",
        "If-None-Match": "1"
    },
    credentials: 'include'
}).then(response => {
    if (response.status === 302) {
        const redirectUrl = response.headers.get('Location');
        console.log('Redirecting to:', redirectUrl);
        window.location.href = redirectUrl;
    } else if (response.status === 304) {
        console.log(response)
        console.log('Resource not modified, using cached version.');
    } else {
        const etag = response.headers.get('ETag');
        console.log('ETag:', etag);
        return response.json();
    }
}).then(data => {
    if (data !== null) {
        console.log(data);
    }
}).catch(error => {
    console.error('Fetch error:', error);
});
```

**Node:**

`credentials: 'include'` is a configuration option in the Fetch API used to control whether credentials information (such as Cookies, HTTP authentication, etc.) should be included in the request. Setting it to `'include'` means that the request will include credentials information, allowing cross-origin requests to carry Cookies.

In cross-origin requests, by default, browsers do not send requests with credentials information (such as Cookies). However, if the server response includes appropriate CORS headers (such as `Access-Control-Allow-Credentials: true`), and the client request includes the configuration `credentials: 'include'`, the browser will include credentials information in the request.

When using `credentials: 'include'`, it is important to ensure that the server is correctly configured with CORS headers to allow requests with credentials. This is done to ensure security and prevent potential Cross-Site Request Forgery (CSRF) attacks.



**set cookie:**

```js
res.cookie('access-token', 'dream-legacy', { httpOnly: true, secure: true, sameSite: 'None' });
```



**Expose Header:**

```js
res.setHeader('Access-Control-Expose-Headers', 'Date');
```

**Node:**

Earlier, we haven't been handling headers, but in reality, we need to add this property for us to successfully obtain it.



### Independent task:

Summarize the mentioned content.

**app.js**

```js
const bodyParser = require('body-parser');
const cors = require('cors');
const express = require('express');
const http = require('http');

// --------------------Configuration-------------------- //
const app = express();
const router = express.Router();

app.use(bodyParser.json());

// ==================== CORS MiddleWare ====================
// const corsOptions = {
//     origin: 'https://www.google.com',
//     credentials: true,
// };
// app.use(cors(corsOptions));

// --------------------Routes-------------------- //
const intermediateRouter = require('./intermediate-router.js');
app.use('', intermediateRouter);

// --------------------Create Server-------------------- //
const port = 4500;
http.createServer(app).listen(port, '0.0.0.0', () => {
    console.log(`Server start on port ${port}`);
});
```

**fetch.js**

```js
const redirectTarget = "https://www.baidu.com/";

const apiUrl = `http://localhost:4500/api/example?redirect=${encodeURIComponent(redirectTarget)}`;

var headersInfo = {};

var getResponse = async function (testCase) {
    let apiUrl = 'http://localhost:4500/api/example';
    let redirectTarget = "https://www.baidu.com/";
    let fetchHeaders = {};
    let method = 'GET';

    if (testCase === 200) {
        fetchHeaders = {
            "Content-Type": "application/json",
        }
    } else if (testCase === 302) {
        apiUrl = `http://localhost:4500/api/example?redirect=${encodeURIComponent(redirectTarget)}`;
    } else if (testCase === 304) {
        fetchHeaders = {
            "Content-Type": "application/json",
            "If-None-Match": "da6abab4563a9682dca2b2312b878acb"
        }
    } else if (testCase === 'download') {
        fetchHeaders = {
            "Content-Type": "text/plain",
            "DownLoad": true
        }
    }

    try {
        const response = await fetch(apiUrl, {
            method: method,
            headers: fetchHeaders,
            credentials: 'include'
        })
        headersInfo.Server = response.headers.get('Server');
        if (response.status === 200) {
            // Please according to Content-Type get response
            const res = await response;
            console.log("status: 200", res);
        } else if (response.status === 302) {
            headersInfo.Location = response.headers.get('Location');
            console.log("status: 302", headersInfo.Location);
            // window.location.href = headersInfo.Location;
        } else if (response.status === 304) {
            console.log("status: 304")
        }

    } catch (error) {
        console.error('Fetch error:', error);
    }
}

getResponse(200);
```

**intermediate-router.js**

```js
const express = require('express');
const crypto = require('crypto');
const intermediateRouter = express.Router();

intermediateRouter.use((req, res, next) => {
    // Access-Control-Allow
    res.setHeader('Access-Control-Allow-Origin', 'https://www.google.com');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Referer, Sec-Ch-Ua, Sec-Ch-Ua-Mobile, Sec-Ch-Ua-Platform, User-Agent');
    res.setHeader('Access-Control-Allow-Methods', '*');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Max-Age', '3600');

    // Expose Header
    res.setHeader('Access-Control-Expose-Headers', 'Server, Location');

    // X-Custom-Header
    res.setHeader('X-Custom-Header', 'Custom-Value');

    // Server
    res.setHeader('Server', "VL's Mock Server");

    // Date
    res.setHeader('Date', new Date().toUTCString());

    // Set-Cookie
    res.cookie('access-token', 'dream-legacy', { httpOnly: true, secure: true, sameSite: 'None' });

    // Cache-Control
    if (req.method === 'GET') {
        res.setHeader('Cache-Control', 'public, max-age=3600');
    } else {
        res.setHeader('Cache-Control', 'no-store');
    }

    next();
});

intermediateRouter.route('/api/example')
    .get((req, res) => {
        const responseData = { message: 'GET request handled' };

        // Content-Length
        res.setHeader('Content-Length', Buffer.byteLength(JSON.stringify(responseData)));

        // ETag
        const etagContent = "VL's Mock Server";
        const etag = crypto.createHash('md5').update(etagContent).digest('hex');
        const ifNoneMatch = req.headers['if-none-match'];

        // redirect
        const redirectTarget = req.query.redirect;

        // download
        const download = req.headers['download'];

        if (ifNoneMatch === etag) {
            res.status(304).end();
        } else if (redirectTarget) {
            // Location
            res.redirect(302, redirectTarget);
            // ==================== Equivalent result as the code below ====================
            // res.setHeader('Location', redirectTarget);
            // res.status(302).end();
        } else if (download === 'true') {
            // Content-Disposition
            res.setHeader('Content-Disposition', 'attachment; filename="example.txt"');

            res.setHeader('Content-Type', 'text/plain');

            const responseData = 'This is the content of the example file.';
            res.status(200).send(responseData);
        } else {
            res.setHeader('Content-Type', 'application/json');
            res.status(200).json(responseData);
        }
    })
    .post((req, res) => {
        const responseData = { message: 'POST request handled' };
        res.status(200).json(responseData);
    })

module.exports = intermediateRouter;
```

This is my summary, what about yours?
