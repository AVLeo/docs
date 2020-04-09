# ATS Apache traffic server
## ats plugin
1. http hook
```
TS_HTTP_CACHE_LOOKUP_COMPLETE_HOOK
Called after the HTTP state machine has completed the cache lookup for the document requested in the ongoing transaction. Register this hook via TSHttpTxnHookAdd or TSHttpHookAdd. Corresponds to the event TS_EVENT_HTTP_CACHE_LOOKUP_COMPLETE.

TS_HTTP_OS_DNS_HOOK
Called immediately after the HTTP state machine has completed a DNS lookup of the origin server. The HTTP state machine will know the origin server’s IP address at this point, which is useful for performing both authentication and blacklisting. Corresponds to the event TS_EVENT_HTTP_OS_DNS.

TS_HTTP_POST_REMAP_HOOK
Called immediately after remapping occurs, before cache lookup. Corresponds to the event TS_EVENT_HTTP_POST_REMAP.

TS_HTTP_PRE_REMAP_HOOK
Called after the request header is read from the client, before any remapping of the headers occurs. Corresponds to the event TS_EVENT_HTTP_PRE_REMAP.

TS_HTTP_READ_CACHE_HDR_HOOK
Called immediately after the request and response header of a previously-cached object is read from cache. This hook is only called if the document is being served from cache. Corresponds to the event TS_EVENT_HTTP_READ_CACHE_HDR.

TS_HTTP_READ_RESPONSE_HDR_HOOK
Called immediately after the response header is read from the origin server or parent proxy. Corresponds to the event TS_EVENT_HTTP_READ_RESPONSE_HDR.

TS_HTTP_RESPONSE_TRANSFORM_HOOK
See “Transformations” for information about transformation hooks.

TS_HTTP_READ_REQUEST_HDR_HOOK
Called immediately after the request header is read from the client. Corresponds to the event TS_EVENT_HTTP_READ_REQUEST_HDR.

TS_HTTP_REQUEST_TRANSFORM_HOOK
See “Transformations” for information about transformation hooks.

TS_HTTP_SELECT_ALT_HOOK
See “HTTP Alternate Selection” for information about the alternate selection mechanism.

TS_HTTP_SEND_RESPONSE_HDR_HOOK
Called immediately before the proxy’s response header is written to the client; this hook is usually used for modifying the response header. Corresponds to the event TS_EVENT_HTTP_SEND_RESPONSE_HDR.

TS_HTTP_SEND_REQUEST_HDR_HOOK
Called immediately before the proxy’s request header is sent to the origin server or the parent proxy. This hook is not called if the document is being served from cache. This hook is usually used for modifying the proxy’s request header before it is sent to the origin server or parent proxy.

Caution:
TS_HTTP_SEND_REQUEST_HDR_HOOK may callback several times when the OS crashed. Be careful to use functions such as TSContDestroy in TS_HTTP_SEND_REQUEST_HDR_HOOK hook.

TS_HTTP_SSN_CLOSE_HOOK
Called when an HTTP session ends. A session ends when the client connection is closed. You can only add this hook as a global hook

TS_HTTP_SSN_START_HOOK
Called when an HTTP session is started. A session starts when a client connects to Traffic Server. You can only add this hook as a global hook.

TS_HTTP_TXN_CLOSE_HOOK
Called when an HTTP transaction ends.

TS_HTTP_TXN_START_HOOK
Called when an HTTP transaction is started. A transaction starts when either a client connects to Traffic Server and data is available on the connection, or a previous client connection that was left open for keep alive has new data available.
```

## record config
1. raw cache
```
Traffic Server maintains a small RAM cache of extremely popular objects. This RAM cache serves the most popular objects as quickly as possible and reduces load on disks, especially during temporary traffic peaks. You can configure the RAM cache size to suit your needs, as described in Changing the Size of the RAM Cache below.

The RAM cache supports two cache eviction algorithms, a regular LRU (Least Recently Used) and the more advanced CLFUS (Clocked Least Frequently Used by Size; which balances recentness, frequency, and size to maximize hit rate, similar to a most frequently used algorithm). The default is to use LRU, and this is controlled via proxy.config.cache.ram_cache.algorithm.

Both the LRU and CLFUS RAM caches support a configuration to increase scan resistance. In a typical LRU, if you request all possible objects in sequence, you will effectively churn the cache on every request. The option proxy.config.cache.ram_cache.use_seen_filter can be set to add some resistance against this problem.

In addition, CLFUS also supports compressing in the RAM cache itself. This can be useful for content which is not compressed by itself (e.g. images). This should not be confused with Content-Encoding: gzip, this feature is only present to save space internally in the RAM cache itself. As such, it is completely transparent to the User-Agent. The RAM cache compression is enabled with the option proxy.config.cache.ram_cache.compress.
```
2. cache inspector utility
```
Inspecting the Cache
Traffic Server provides a Cache Inspector utility that enables you to view, delete, and invalidate URLs in the cache (HTTP only). The Cache Inspector utility is a powerful tool that’s capable of deleting all the objects in your cache. Therefore, make sure that only authorized administrators are allowed to access this utility through proper use of the @src_ip option in remap.config and the instructions detailed in Controlling Access.

Accessing the Cache Inspector Utility
To access the Cache Inspector utility:

Set proxy.config.http_ui_enabled to 1.

To access the cache inspector in reverse proxy mode, you must add a remap rule to remap.config to expose the URL. This should be restricted to a limited set of hosts using the @src_ip option. To restrict access to the network 172.28.56.0/24, use

map http://yourhost.com:8080/myCI/ http://{cache} @action=allow @src_ip=172.28.56.1-172.28.56.254
Reload the Traffic Server configuration by running traffic_ctl config reload.

Open your web browser and go to the the following URL:

http://yourhost.com:8080/myCI/
You will now be presented with the Cache Inspector interface.

Using the Cache Inspector Utility
The Cache Inspector Utility provides several options that enable you to view and delete the contents of your cache.

Lookup URL
Search for a particular URL in the cache. When Traffic Server finds the URL in the cache, it will display details of the object that corresponds to the URL (e.g. header length and number of alternates). The option to delete the URL from the cache will be presented.

Delete URL
Delete the object from the cache which corresponds to the given URL. Success or failure will be indicated after a delete has been attempted.

Regex Lookup
Search URLs within the cache using one or more regular expressions.

Regex Delete
Deletes all objects from the cache which match the provided regular expressions.

Regex Invalidate
Marks any objects in the cache which match the given regular expressions as stale. Traffic Server will contact the relevant origin server(s) to confirm the validity and freshness of the cached object, updating the cached object if necessary.
```
3. memory leak
```
# config in records.config, dump memory info every 30 seconds
CONFIG proxy.config.dump_mem_info_frequency INT 30
```