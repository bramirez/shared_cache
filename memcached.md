# Dalli Gem and Memcached #


### Objective ###

* Since MW uses 2 web servers, the application should have a centralized cache


### Solution/Approach ###

* [Dalli Gem](https://github.com/mperham/dalli)
* [Memcached](http://memcached.org/)


### Review of Cache Stores in Rails (3.2.x) ###

1. ActiveSupport::Cache::MemoryStore
   * `config.cache_store = :memory_store, { size: 64.megabytes }`
   * default cache store implementation
   * keeps entries in memory in the same Ruby process
   * not able to share cache data with each other
   * not appropriate for large application deployments
2. ActiveSupport::Cache::FileStore
   * `config.cache_store = :file_store, "/path/to/cache/directory"`
   * uses the file system to store entries
   * could share a cache by using a shared file system, but not ideal and is not recommended
   * appropriate for low to medium traffic sites that are served off one or two hosts
3. ActiveSupport::Cache::MemCacheStore
   * `config.cache_store = :mem_cache_store, "cache-1.example.com", "cache-2.example.com"`
   * uses Danga's memcached server to provide a centralized cache for your application
   * uses the bundled [memcache-client](https://github.com/mperham/memcache-client) gem by default
   * most popular cache store for production websites
4. ActiveSupport::Cache::EhcacheStore
   * `config.cache_store = :ehcache_store`
   * used for JRuby apps
   * an open source Java cache
5. ActiveSupport::Cache::NullStore
  * `config.cache_store = :null_store`
  * meant to be used only in development or test environments and it never stores anything


### Dalli Gem ###
* since memcache-client is deprecated and no longer supported
* default in Rails 4
* https://github.com/mperham/dalli

###### Configuration ######
`config/environments/production.rb`:
   * `config.cache_store = :dalli_store, '10.180.31.55:11211'`


### Memcached ###
* `apt-get install memcached`
* comment line `-l 127.0.0.1` in `/etc/memcached.conf`
* Notes: Cache values do not persist on restarts.
