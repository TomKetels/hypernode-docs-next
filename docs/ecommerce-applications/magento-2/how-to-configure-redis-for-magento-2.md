<!-- source: https://support.hypernode.com/en/ecommerce/magento-2/how-to-configure-redis-for-magento-2/ -->
# How to Configure Redis for Magento 2

Redis is a caching method which can increase the speed of the backend and frontend of your shop. On Hypernode every customer has access to Redis cache, starting from 64 MB, depending on the plan. This article will explain how to configure Redis on your Magento 2 shop on Hypernode and how to work with redis-cli.

Want to know how to configure Redis in Magento 1? Have a look at [this article](https://support.hypernode.com/en/ecommerce/magento-1/how-to-configure-redis-for-magento-1)!


Configure Redis Cache for Magento 2
-----------------------------------

There are two ways to configure Redis Cache for Magento 2. You can either run a command which automatically updates the `env.php` with the correct details or you can manually change the `env.php` file.

Configure Redis Cache for Magento 2 Through the Commandline
-----------------------------------------------------------

Use the following command to enable Redis backend caching:

```nginx
cd /data/web/magento2>bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=127.0.0.1 --cache-backend-redis-db=0>
```
Configure Redis Cache for Magento 2 by editing the env.php file
---------------------------------------------------------------

To enable caching in Redis, extend your `/data/web/magento2/app/etc/env.php` with the following snippet. Add this in between the `cache` keys. (Without the `cache` key in the snippet)

```nginx
'cache' => array(> 'frontend' => array(> 'default' => array(> 'backend' => 'Cm_Cache_Backend_Redis',> 'backend_options' => array(> 'server' => '127.0.0.1',> 'port' => '6379',> ),> ),> ),>),
```
>A complete env.php configuration example [can be found over here](https://gist.github.com/hn-support/4bf9575e7896abf57dff2b5ac15f05ef).

Now flush your cache:

```nginx
rm -rf /data/web/magento2/var/cache/*>redis-cli flushall>
```
Configure Redis Full Page Caching for Magento 2
-----------------------------------------------

To enable page caching Redis, extend your /data/web/magento2/app/etc/env.php with the following snippet. >You should paste this in between the cache keys, so leave the cache tag in this snippet out of it.

```nginx
 'cache' => array (> 'frontend' => array (> 'default' => array (> 'backend' => 'Cm_Cache_Backend_Redis',> 'backend_options' => array (> 'server' => '127.0.0.1',> 'port' => '6379',> ),> ),> // Start of snippet> 'page_cache' => array (> 'backend' => 'Cm_Cache_Backend_Redis',> 'backend_options' => array (> 'server' => '127.0.0.1',> 'port' => '6379',> 'database' => '1',> 'compress_data' => '0',> ),> ),> // End of snippet> ),> ),
```
>A complete env.php configuration example [can be found over here](https://gist.github.com/hn-support/4bf9575e7896abf57dff2b5ac15f05ef)

>

And flush your cache:

```nginx
rm -rf /data/web/magento2/var/cache/*>redis-cli flushall>
```
Flush Your Caches
-----------------

To flush your Magento cache, clear the Redis database corresponding to your configured Redis database:

```nginx
redis-cli -n $db flushdb>
```
Or alternatively use `n98-magerun2` or the Magento cli tool:

```nginx
## Flush using n98-magerun2>n98-magerun2 cache:flush> >## Flush using magento cli>cd /data/web/magento2 && php bin/magento cache:flush>
```
To flush all sessions, caches etc (flush the full Redis instance), use the following command:

```nginx
redis-cli flushall>
```
Changing the Compression Library
--------------------------------

It is possible to use the compression library 'Snappy' on Hypernode. More information about Snappy can be found in the changelog: [Release-4224](https://changelog.hypernode.com/changelog/release-4224/).

In order to use the compression library Snappy for your Redis cache you need to add `'compression_library' => 'snappy',` in your env.php under:

```nginx
'page_cache' => array (>'backend' => 'Cm_Cache_Backend_Redis',>'backend_options' => array (>
```
Configure Magento 2 to Use Redis as the Session Store
-----------------------------------------------------

You can use Redis for storing sessions too!

Hypernodes bigger than a Grow plan, often have enough memory to store the session data in Redis.
>This way sessions are stored in-memory, making the shop faster and use less IO than when using MySQL or files as session store.

### Configure Magento 2 to Store Sessions in Redis

As Magento 2 is fully supporting Redis, there is no need to install additional extensions to configure Redis. >All you need to do is extend your `app/etc/env.php` and flush your cache.

To enable session storage in Redis, extend your `/data/web/magento2/app/etc/env.php` with the following snippet:

```nginx
'session' => array(
'save' => 'redis',
'redis' => array(
'host' => 'redismaster',
'port' => '6379',
'password' => '',
'timeout' => '2.5',
'persistent_identifier' => '',
'database' => '2',
'compression_threshold' => '2048',
'compression_library' => 'gzip',
'log_level' => '1',
'max_concurrency' => '6',
'break_after_frontend' => '5',
'break_after_adminhtml' => '30',
'first_lifetime' => '600',
'bot_first_lifetime' => '60',
'bot_lifetime' => '7200',
'disable_locking' => '0',
'min_lifetime' => '60',
'max_lifetime' => '2592000',
),
),
```
A complete env.php configuration example [can be found over here](https://gist.github.com/hn-support/4bf9575e7896abf57dff2b5ac15f05ef)

Now flush your cache:

```nginx
rm -rf /data/web/magento2/var/cache/*
redis-cli flushall
```
### Enable Second Redis Instance for Sessions

We have made is possible to enable a second Redis instance more tailored for saving session data (more informatie can be found in our [changelog](https://changelog.hypernode.com/changelog/experimental-changes-redis-sessions-aws-performance/))

To enable the second Redis instance for sessions you run the command: `hypernode-systemctl settings redis_persistent_instance --value True`

After enabling the second Redis instance you need to update the `/data/web/public/app/etc/local.xml` file and change the port value to `6378` instead of the default `6379`. Furthermore you need to add the following line to your crontab:

```
* * * * * redis-cli -p 6378 bgsave
```
### Test Whether Your Sessions Are Stored in Redis

To verify whether your configuration is working properly, first clear your session store:

```nginx
rm /data/web/public/var/sessions/*

```

Now open the site in your browser and hit `F5` a few times or log in to the admin panel. > >If all is well, no additional sessions files should be written to `/data/web/var/sessions`, but instead to the Redis database:

To verify whether your configuration is working properly, first clear your session store:

```nginx
redis-cli -n 2 keys \*

```
Troubleshooting
---------------

A quick note, when you run into the configured max memory limit make sure that the necessary Redis keys are set to volatile (ensure an expire). Otherwise the complete allocated configured memory will fill up and Redis will 'crash'.

When your Redis instance memory is full, and a new write comes in, Redis evicts keys to make room for the write based on your instance's maxmemory policy. This is called the eviction policy.

In some cases we see that when Redis reaches the configured limit and tries to expire keys to make room, the eviction policy gets stuck in a loop. This means keys won’t be expired and Redis reaches its limit.

A temporary solution is to flush the Redis cache, you can do this by using the `flushall` command:

```nginx
redis-cli flushall
```
This will flush out all available Redis databases. Please keep in mind that this is only a temporary solution. The underlying cause is in the code of your application and needs to be permanently resolved.

A more extended how-to about configuring Redis caches can be found on the [Magento help pages](http://devdocs.magento.com/guides/v2.0/config-guide/redis/redis-pg-cache.html).

Bots
----

>As you know, the sessions of your webshop can also be stored in Redis. If you use Redis caching and store the sessions in Redis as well, you'll have to share the available Redis memory. This shouldn't be a problem on a regular basis, however we've seen scenarios wherein a shop stores its sessions in Redis and had some aggressive bots/crawlers visiting the shop. This resulted in a much larger amount of sessions to be stored in Redis than usual which is causing the Redis memory to fill up in no time, and crashes Redis.

You can check the bot traffic on your shop at any time on MageReport. If you want to get a more detailed insight in the bot traffic you can use the command `pnl --yesterday --php --bots --fields ua | sort | uniq -c | sort -n`to get an overview of the top 10 bots that visited your webshop yesterday. For more information about abuse bot check our [article](https://support.hypernode.com/knowledgebase/fixing-bad-performance-caused-by-search-engines/).