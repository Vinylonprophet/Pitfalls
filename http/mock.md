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
app.post('/', (req, res) => {
  res.send('Hello World!');
});
```

You also can define a complex route. It will be mentioned later.

```js
const router = express.Router();
```



#### 4. Listening on port

```js
const port = 4500;
http.createServer(app).listen(port, '0.0.0.0', () => {
    console.log(`Server start on port ${port}`);
});
```



## Enhancing the mock

### Resolving Cross-Origin

