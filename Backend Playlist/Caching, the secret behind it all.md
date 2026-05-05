**Core Concepts of Caching**

**Definition:** Caching is the process of storing a subset of frequently accessed data in a location that is faster to retrieve than the primary storage (0:00 - 1:15).
The Trigger: Caching is used to avoid two main issues: heavy computation (repeatedly performing expensive calculations) and data-intensive operations (sending large amounts of data to many users) (14:13 - 14:48).
Levels of Caching

**Network Level (18:16 - 36:50):**
CDNs (Content Delivery Networks): Uses globally distributed Edge Servers to serve content from locations geographically closer to the end user, significantly reducing buffering for platforms like Netflix (6:11 - 11:14, 19:51 - 26:13).
DNS Caching: Speeds up the process of converting domain names into IP addresses (26:20 - 36:50).
Hardware Level (36:51 - 37:07): Focuses on CPU caches (L1/L2) and the efficiency of sequential data access.
Software/Application Level (18:31 - 18:55): Utilizes in-memory key-value stores like Redis or Memcached to store data directly in RAM for rapid retrieval (37:11 - 42:48).
Key Caching Strategies & Policies

**Caching Strategies:**
Lazy Caching (Cache-aside): Data is cached only when a request is made for it (43:00 - 44:10).
Write-Through Caching: Data is updated in the database and the cache simultaneously, ensuring the cache remains fresh (44:11 - 45:18).
Eviction Policies: Since memory is limited, these policies determine what to delete when the cache is full:
LRU (Least Recently Used): Removes the item that hasn't been accessed for the longest time (47:10 - 48:23).
LFU (Least Frequently Used): Removes the item with the lowest access frequency (48:25 - 49:38).
TTL (Time To Live): Automatically expires keys based on a predefined duration (49:41 - 50:23).

**Real-World Applications**
Database Query Optimization: Caching the results of complex, compute-intensive SQL queries (51:30 - 54:30).
User Sessions: Storing session tokens in Redis to verify users instantly without hitting the main database (55:51 - 56:57).
Rate Limiting: Managing API request limits efficiently to prevent database flooding (1:02:52 - 1:03:28).
