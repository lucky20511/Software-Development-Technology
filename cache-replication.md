# Cache Replication

With replication, data is generated one time and copied or replicated to other servers in the cluster, saving time and resources. Caching in a cluster has additional concerns. In particular, the same data can be required and generated in multiple places. Also, the permission the resources need to generate the cached data can be restricted, preventing access to the data.

Cache replication deals with these concerns by generating the data one time and copying it to the other servers in the cluster. It also aids in cache consistency. Cache entries that are not needed are removed or replaced.

The data replication configuration can exist as part of the web container dynamic cache configuration accessible through the administrative console, or on a per cache entry basis through thecachespec.xmlfile. With thecachespec.xmlfile, you can configure cache replication at the web container level, but disable it for a specific cache entry.

Cache replication can take on three forms:

* PUSH
  - Send out new entries, both ID and data, and updates to those entries.
* PULL
  - Requests data from other servers in the cluster when that data is not locally present. This mode of replication is not recommended.
* PUSH/PULL
  - Sends out IDs for new entries, then, only requests from other servers in the cluster entries for IDs previously broadcast. The dynamic cache always sends out cache entry invalidations.

You can also perform a batch update. Specifically, for PUSH or PUSH/PULL, the dynamic cache broadcasts the update asynchronously, based on a timed interval rather than sending them immediately when they are created. Invalidations are sent immediately. Distribution of invalidations prevents stale data from residing in a cluster. For more information about configuring cache replication, see [Configuring cache replication](https://www.ibm.com/support/knowledgecenter/SSAW57_8.0.0/com.ibm.websphere.nd.doc/info/ae/ae/tdyn_cachereplication.html?view=kc) and [Dynamic cache service settings](https://www.ibm.com/support/knowledgecenter/SSAW57_8.0.0/com.ibm.websphere.nd.doc/info/ae/ae/udyn_rcachesettings.html?view=kc).

In PUSH/PULL mode, the cached object is kept locally on the server that creates it; however, other servers also use the cache ID and store it in the DRSPushPullTable table. If a remote server needs the object, it requests the object by cache ID, or name, from the creating server. Each cache instance has one DRSPushPullTable table that is associated with it. The following conditions cause the DRSPushPullTable table to grow too big:

* There are too many entries being shared with other servers.
* Not many entries are expiring.
* If you are using the disk offload feature, the disk scan is not running often to evict the expired entries.

Use the following suggestions to resolve the issue:

* Increase the heap size to 1.5 GB or 2 GB, if possible.
* Maintain better distribution for the expiration time of entries, for example:
  * 20% of the entries never expire.
  * 30% of the entries expire in 3600 seconds.
  * 30% of the entries expire in 600 seconds.
  * 20% of the entries expire in 60 seconds.
* When you use the disk offload feature in WebSphere® Application Server 7.0, for the disk cache performance settings, which are low, balanced, and custom, adjust the disk cleanup frequency to an optimal value, in minutes. For example, about 20% of the entries expire at that time.



