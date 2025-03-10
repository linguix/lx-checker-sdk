# Proxy Server for Linguix Checker SDK

This document explains how to set up and use a proxy server with the Linguix Checker SDK to securely hide your API keys from client-side code.

## Why Use a Proxy Server?

When integrating the Linguix Checker SDK into client-side applications, you might not want to expose your API key directly in the client code. A proxy server allows you to:

1. Keep your API key secure on your server
2. Add custom authentication for your users
3. Control and monitor API usage
4. Add additional business logic or rate limiting

## Proxy Server Example

Below is a simple example of a Node.js proxy server that forwards requests to the Linguix API while handling authentication and API key management:

```javascript
const http = require('http');
const httpProxy = require('http-proxy');
const { URL } = require('url');

const PORT = 3000;
const TARGET = 'https://api.linguix.com'; // Linguix API server URL
const API_KEY = 'your-api-key'; // Replace with your actual API key

// Create a proxy server with WebSocket support
const proxy = httpProxy.createProxyServer({
    target: TARGET,
    ws: true,
    changeOrigin: true
});

// Global error handler for proxy events (HTTP requests)
proxy.on('error', (err, req, res) => {
    console.error('Proxy error:', err);
    if (res && !res.headersSent) {
        res.writeHead(500, { 'Content-Type': 'text/plain' });
    }
    if (res) {
        res.end('Something went wrong.');
    }
});

// This function authenticates the request and rewrites the URL query string.
// It expects a "token" parameter for authenticating the user. On success, it removes
// any client-provided token/apiKey and appends your server's API key.
function authenticateAndModify(req, res) {
    try {
        // Construct a URL object using a dummy base (since req.url is relative)
        const urlObj = new URL(req.url, 'http://dummy');
        
        // Extract the token from query parameters
        const token = urlObj.searchParams.get('clientToken');
        console.log('Authentication attempt with token:', token);
        
        // Simple authentication check: token must be non-empty.
        // You can expand this to check against a database or other auth service.
        if (!token || token.trim() === '' || token === 'undefined') {
            if (res) {
                res.writeHead(401, { 'Content-Type': 'text/plain' });
                res.end('Authentication failed: Invalid or missing token');
            }
            return false;
        }

        // At this point, authentication succeeded.
        // Remove any client-provided token or apiKey and insert your API key.
        urlObj.searchParams.delete('clientToken');
        urlObj.searchParams.delete('apiKey');
        urlObj.searchParams.append('apiKey', API_KEY);

        // Rewrite the request URL with the updated query string.
        req.url = urlObj.pathname + urlObj.search;
        console.log('Modified URL:', req.url);

        return true;
    } catch (err) {
        console.error('Error during authentication:', err);
        if (res) {
            res.writeHead(500, { 'Content-Type': 'text/plain' });
            res.end('Internal server error');
        }
        return false;
    }
}

// Create an HTTP server that handles both HTTP requests and WebSocket upgrades.
const server = http.createServer((req, res) => {
    // For standard HTTP requests, authenticate and modify the URL.
    if (!authenticateAndModify(req, res)) {
        return;
    }
    // Forward the HTTP request to the target server.
    proxy.web(req, res);
});

// Handle WebSocket upgrade requests.
server.on('upgrade', (req, socket, head) => {
    // For upgrade requests, run authentication.
    if (!authenticateAndModify(req)) {
        // If authentication fails, close the socket.
        socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n');
        socket.destroy();
        return;
    }
    socket.on('error', (err) => {
        console.error('Socket error:', err);
    });
    // Forward the WebSocket connection.
    proxy.ws(req, socket, head);
});

server.listen(PORT, () => {
    console.log(`Proxy server with authentication is running on port ${PORT}`);
});
```

## Setting Up the Proxy Server

1. Create a new Node.js project
2. Install the required dependencies:
   ```bash
   npm install http-proxy
   ```
3. Save the above code to a file (e.g., `proxy-server.js`)
4. Replace `'your-api-key'` with your actual Linguix API key
5. Start the server:
   ```bash
   node proxy-server.js
   ```

## Configuring the SDK to Use Your Proxy

When initializing the Linguix Checker SDK in your client application, you'll need to configure it to use your proxy server instead of directly connecting to the Linguix API:

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

// Configure the SDK to use your proxy server
LinguixCheckerSDK.initializeWithConfig(
    {
        url: 'http://localhost:3000',
        options: {
            query: {
                clientToken: 'some-token'
            }
        }
    });
```

## Troubleshooting

If you encounter issues with your proxy server:

1. Check the server logs for error messages
2. Verify that your proxy server is correctly forwarding both HTTP and WebSocket connections
3. Ensure your client is correctly configured to use the proxy server
4. Test the connection using tools like cURL or Postman

For additional support, contact us at hi@linguix.com.
