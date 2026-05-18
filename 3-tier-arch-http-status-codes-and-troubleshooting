# 3-Tier Architecture HTTP Status Codes Deep-Dive

This repository serves as an operational playbook for managing and troubleshooting HTTP status codes within a standard 3-tier architecture (**Nginx Frontend/LB -> Node.js App -> MySQL Database**).

---

## 🗺️ High-Level Status Code Overview

### 🟢 2XX Success Codes (Infrastructure Healthy)
*   **200 OK**: Successful `GET` requests. Node.js fetched data from MySQL, and Nginx delivered it cleanly.
*   **201 Created**: Successful `POST` or `PUT` requests. A brand-new record was successfully written into MySQL.
*   **204 No Content**: Successful `DELETE` or empty requests. The database action succeeded, but there is no body data to return.

### 🟡 3XX Redirectional Codes (Traffic Routing)
*   **301 Moved Permanently**: Nginx-level structural route change (e.g., forcing HTTP traffic over to secure HTTPS).
*   **304 Not Modified**: Network bandwidth saver. Nginx or Node.js tells the browser to load data from its local cache.

### 🟠 4XX Client-Side Errors (Bad Client Requests)
*   **401 Unauthorized**: Missing Identity. The client forgot to provide an API key or Token header.
*   **403 Forbidden**: Missing Permissions. The server knows who the client is, but their role is blocked from this resource.
*   **404 Not Found**: Missing Route. The client mistyped the URL, or Nginx lacks a matching routing block.
*   **405 Method Not Allowed**: Wrong HTTP Verb. The client used an unsupported method (e.g., `GET` on a `POST`-only route).

### 🔴 5XX Server-Side Errors (Infrastructure & Code Failures)
*   **500 Internal Server Error**: Application Code Crash. Node.js threw an unhandled exception or hit a raw MySQL syntax error.
*   **502 Bad Gateway**: App Process Offline. Nginx is running, but the Node.js application process is completely crashed or down.
*   **503 Service Unavailable**: Infrastructure Overload. Nginx is dropping traffic due to capacity limits or maintenance.
*   **504 Gateway Timeout**: Database/Network Bottleneck. Node.js timed out waiting for a locked or unindexed MySQL query.

---

## 🔍 DevOps Troubleshooting & Log Matrix

When an alert triggers, use this matrix to determine whether the root cause is an **Infrastructure Configuration issue** or an **Application/Database Bug**, and locate the correct logs immediately.


| Status Code | Primary Log Target | What to Look For in the Logs | Responsible Team |
| :--- | :--- | :--- | :--- |
| **301 / 304** | `/var/log/nginx/access.log` | Verify redirect rules or cache hit/miss headers (`X-Cache-Status`). | DevOps / SRE |
| **401 / 403** | Node.js Application Logs | Check validation token parsing logic or user role evaluation blocks. | Backend Devs |
| **404** | `/var/log/nginx/error.log` | Look for `test-upstream not found` or generic file routing errors. | DevOps |
| **405** | Node.js Application Logs | Verify Express router configurations (`app.post` vs `app.get`). | Backend Devs |
| **500** | Node.js Application Logs | Locate unhandled promise rejections or raw MySQL query syntax failures. | Backend Devs |
| **502** | `/var/log/nginx/error.log` | Look for `connect() failed (111: Connection refused)` while connecting to upstream. | DevOps |
| **503** | `/var/log/nginx/error.log` | Search for connection throttling, worker connections limits, or OOM drops. | DevOps |
| **504** | `/var/log/nginx/error.log` | Look for `upstream timed out (110: Connection timed out)` alerts. Check MySQL slow query logs. | DevOps & DBAs |

---

## 🛠️ How to Debug and Inspect Logs

### 1. Stream Live Nginx Ingress Traffic
```bash
tail -f /var/log/nginx/access.log | grep -E "404|502|504"
```

### 2. Check for Process Failures (For 502 Errors)
```bash
# If using systemd/PM2
pm2 status
journalctl -u nodejs-app -n 50 --no-pager
```

### 3. Inspect Upstream Timeouts (For 504 Errors)
```bash
grep "timed out" /var/log/nginx/error.log
```
