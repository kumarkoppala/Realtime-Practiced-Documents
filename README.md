# Realtime-Practiced-Documents
# 3-Tier Architecture HTTP Status Codes Deep-Dive

This repository implements a 3-tier architecture (**Nginx Frontend/LB -> Node.js App -> MySQL Database**) to demonstrate how common HTTP status codes map to specific real-world scenarios across the stack.

## 🗺️ Architecture Overview
*   **Frontend / Load Balancer:** Nginx (Port 80)
*   **Application Layer:** Node.js Express (Port 8080)
*   **Database Layer:** MySQL (Port 3306)

---

## 🚦 HTTP Status Codes Deep-Dive

### 🟢 2XX Success Codes
*   **200 OK**: The Node.js app successfully queried MySQL, fetched the transaction data, and Nginx delivered it back to the client.
*   **201 Created**: Occurs when a `POST` or `PUT` request successfully inserts or creates a brand-new transaction record in the MySQL database.
*   **204 No Content**: Node.js and MySQL processed a request successfully (like a successful `DELETE` or an empty `GET` result), but there is no body data to return.

### 🟡 3XX Redirectional Codes
*   **301 Moved Permanently**: Nginx is forcing a hardware/protocol redirect. For example, routing insecure HTTP traffic over to HTTPS.
*   **304 Not Modified**: Nginx or Node.js utilizes conditional headers (`If-None-Match`/ETags) to tell the browser the data hasn't changed, saving bandwidth by loading it from the local cache.

### 🟠 4XX Client-Side Error Codes
*   **401 Unauthorized**: The request is missing a valid authorization token or API key in the headers. Node.js rejects it because identity is unverified.
  ```http
Example:
  GET /api/transaction HTTP/1.1
Host: 98.88.35.125
User-Agent: curl/7.68.0
Accept: */*

Fix: We need to provide authorization
curl -X GET http://98.88.35 \
     -H "Authorization: Bearer your-secret-json-web-token-here"

```

*   **403 Forbidden**: Node.js validated the user's identity, but their assigned role lacks permissions to access the specific transaction route.
*   **404 Not Found**: The endpoint path was mistyped (e.g., `/api/transactions`), or Nginx does not have a matching location block routing the path to the backend.
*   **405 Method Not Allowed**: The route exists, but you sent an unsupported HTTP method (e.g., executing a `GET` on an endpoint configured exclusively for `POST`).

### 🔴 5XX Server-Side Error Codes
*   **500 Internal Server Error**: A raw database syntax error or unhandled code exception occurred inside Node.js, crashing the active request threads.
*   **502 Bad Gateway**: Nginx is healthy, but the underlying Node.js application process is entirely down, crashed, or not running.
*   **503 Service Unavailable**: Nginx is dropping connections due to traffic overload, or the backend service is temporarily offline for maintenance.
*   **504 Gateway Timeout**: Nginx reached Node.js, but Node.js failed to reply within the timeout threshold. Usually caused by unindexed, locked, or slow MySQL queries.
