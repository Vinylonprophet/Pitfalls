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

