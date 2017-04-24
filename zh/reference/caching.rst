23. 缓存
=========

Doctrine 在 Common 包下面提供了一些缓存驱动并且实现了一些流行的缓存，如：APC, Memcache, Xcache。也提供 ArrayCache 驱动，ArrayCache 驱动存储在 php 的数组里面。显然，缓存不会在不同的请求中存在。但是对于开发环境测试非常有用。

23.1. 缓存驱动
--------------

缓存驱动的接口定义在 ``Doctrine\Common\Cache\Cache``。所有的缓存驱动继承类 ``Doctrine\Common\Cache\AbstractCache``，该类实现了 Cache 接口。

接口定义了下面的方法供你实现：

-  fetch($id) - 从缓存中获得一条记录
-  contains($id) - 判断缓存中是否存在记录
-  save($id, $data, $lifeTime = false) - 将数据存入缓存 x 秒。 0 = 不限制时间
-  delete($id) - 删除缓存

每一个驱动都需要继承 AbstractCache 类并且实现该类定义的一些受保护的抽象方法：

-  \_doFetch($id)
-  \_doContains($id)
-  \_doSave($id, $data, $lifeTime = false)
-  \_doDelete($id)

公共的方法 ``fetch()``, ``contains()`` 等利用上面受保护的方法去实现，这样的代码组织方式使得驱动里的受保护方法可以做最原始的缓存实现。``AbstrachCache`` 类可以定制上面那些方法的功能。

本文档不包括 Doctrine 附带的每个缓存驱动器。有关最新的列表，请参阅 GitHub 上的缓存目录 `<https://github.com/doctrine/cache/tree/master/lib/Doctrine/Common/Cache>`。

23.1.1. APC
-------------

为了使用 APC 缓存驱动你必须编译和在 php.ini 中激活它，你可以阅读这篇文章 `in the PHP Documentation <http://us2.php.net/apc>`_ ，他可以提供一些背景知识让你怎么去安装和使用。

下面是一个简单的例子，说明了怎么去使用 APC 缓存。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\ApcCache();
    $cacheDriver->save('cache_id', 'my_data');

23.1.2. Memcache
-------------------

为了使用 Memcache 缓存，你必须去编译和在 php.ini 中激活它，你可以在 `php 官网  <http://php.net/memcache>`_ 上了解 memcache 的相关信息。

下面是一个简单的例子怎么去使用 memcache 缓存。

.. code-block:: php

    <?php
    $memcache = new Memcache();
    $memcache->connect('memcache_host', 11211);
    
    $cacheDriver = new \Doctrine\Common\Cache\MemcacheCache();
    $cacheDriver->setMemcache($memcache);
    $cacheDriver->save('cache_id', 'my_data');

23.1.3. Memcached
--------------------

Memcached 是一个最新最完整的 Memcache 扩展。

为了使用Memcached缓存你必须编译和在php.ini中激活它，你可以在 `php 官网 <http://php.net/memcached>`_上了解memcache的相关信息， 如何安装和使用。

下面是一个简单的例子告诉你如何去使用 Memcached 缓存。

.. code-block:: php

    <?php
    $memcached = new Memcached();
    $memcached->addServer('memcache_host', 11211);
    
    $cacheDriver = new \Doctrine\Common\Cache\MemcachedCache();
    $cacheDriver->setMemcached($memcached);
    $cacheDriver->save('cache_id', 'my_data');

23.1.4. Xcache
--------------------

为了使用 Xcache 缓存，你必须编译和在 php.ini 中激活它，你可以在 `这儿 <http://xcache.lighttpd.net/>`_ 了解 Xcache 的背景知识和如何去安装使用它。

下面是一个简单的例子如何使用 Xcache。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\XcacheCache();
    $cacheDriver->save('cache_id', 'my_data');

23.1.5. Redis
--------------------

为了使用 readis 缓存，你必须编译然后在 php.ini 中激活它，你可以从 `这里 <http://redis.io/>`_ 阅读 Redis 的相关内容。也可以从 `这里 <https://github.com/nicolasff/phpredis/>`_ 了解如何使用和安装 PHP Redis 扩展。

下面是一个简单的例子如何使用redis缓存。

.. code-block:: php

    <?php
    $redis = new Redis();
    $redis->connect('redis_host', 6379);

    $cacheDriver = new \Doctrine\Common\Cache\RedisCache();
    $cacheDriver->setRedis($redis);
    $cacheDriver->save('cache_id', 'my_data');

23.2. 使用缓存驱动
-------------------

这个章节中我们描述了如何利用缓存驱动的 API 去保存缓存，检查缓存是否存在，获得缓存数据，删除缓存数据。在我们的例子中将使用 ``ArrayCache``。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\ArrayCache();

23.2.1. 保存缓存
-------------------

保存数据到缓存中使用 ``save()`` 方法

.. code-block:: php

    <?php
    $cacheDriver->save('cache_id', 'my_data');

``save()`` 方法接受三个参数：

-  ``$id`` - 缓存 id
-  ``$data`` - 缓存数据
-  ``$lifeTime`` - 缓存的过期时间

你可以保存任意类型数据，string，array，object 等等。

.. code-block:: php

    <?php
    $array = array(
        'key1' => 'value1',
        'key2' => 'value2'
    );
    $cacheDriver->save('my_array', $array);

23.2.2. 检查缓存
-------------------

检查缓存是否存在非常简单，使用 ``contains()`` 方法，他接受一个缓存 ID 参数

.. code-block:: php

    <?php
    if ($cacheDriver->contains('cache_id')) {
        echo 'cache exists';
    } else {
        echo 'cache does not exist';
    }

23.2.3. 获取缓存
-------------------

如果你想获得缓存数据你可以使用 ``fetch()`` 方法，它像 ``contains()`` 方法一样，接受一个缓存 ID 参数。

.. code-block:: php

    <?php
    $array = $cacheDriver->fetch('my_array');

23.2.4. 删除缓存
-------------------

就像是想象中那样，删除跟保存，检查和获得一样简单。我们有很多方法可以去删除缓存数据，你可以通过 ID，正则表达式，前缀，后缀，或者你可以删除所有的数据。

23.2.4.1. 通过缓存 ID 删除缓存
----------------------------

.. code-block:: php

    <?php
    $cacheDriver->delete('my_array');

23.2.4.2. 删除所有缓存
----------------------

如果你想删除所有的缓存你可以使用 ``deleteAll()`` 方法。

.. code-block:: php

    <?php
    $deleted = $cacheDriver->deleteAll();

24.2.5. 命名空间
----------------------

如果你的应用程序大量的使用缓存和在你的应用程序的多个部分利用缓存，或者不同的应用程序在一台服务器上面，你可能会遇到缓存名重复的问题，你可以使用命名空间，可以给一个缓存驱动通过 ``setNamespace()`` 方法设置命名空间。

.. code-block:: php

    <?php
    $cacheDriver->setNamespace('my_namespace_');

23.3. 集成 ORM
------------------------

Doctrine 的 ORM 包已经集成了缓存驱动，允许你改善 Doctrine 很多方面的性能。你只需要做一些简单的配置和方法的调用。

23.3.1. 查询缓存
----------------------

强烈建立在生成环境缓存 DQL 生成的 sql 语句，在你改变 DQL 查询前，再通过大量的时间去解析 DQL 语句没有任何意义。

可用通过下面的配置去缓存查询语句。

.. code-block:: php

    <?php
    $config = new \Doctrine\ORM\Configuration();
    $config->setQueryCacheImpl(new \Doctrine\Common\Cache\ApcCache());

23.3.2. 结果缓存
----------------------

将查询的结果缓存起来我们不需要再去数据库里面进行查询了。你需要做一些配置去实现结果缓存。

.. code-block:: php

    <?php
    $config->setResultCacheImpl(new \Doctrine\Common\Cache\ApcCache());

现在当你执行 DQL 查询时，你可以配置它进行查询结果缓存。

.. code-block:: php

    <?php
    $query = $em->createQuery('select u from \Entities\User u');
    $query->useResultCache(true);

你也可以配置个别的查询使用不同的缓存驱动。

.. code-block:: php

    <?php
    $query->setResultCacheDriver(new \Doctrine\Common\Cache\ApcCache());

.. note::

    在查询上面设置了缓存驱动将自动的激活查询结果进行缓存。如果你想要禁用，那么传递 ``false`` 到 ``useResultCache()`` 方法中。

    ::

        <?php
        $query->useResultCache(false);


如果你想设置缓存时间，你可以使用 ``setResultCacheLifetime()`` 方法。

.. code-block:: php

    <?php
    $query->setResultCacheLifetime(3600);

如果你不在 ``setResultCacheId()`` 方法中提供 ID，那么 doctrine 会自动的生成一个 hash 的值。

.. code-block:: php

    <?php
    $query->setResultCacheId('my_custom_id');

你也可以通过设置 ``useResultCache()`` 方法的第二和第三个参数设置缓存生存时间和缓存 ID。

.. code-block:: php

    <?php
    $query->useResultCache(true, 3600, 'my_custom_id');

23.3.3. Metadata 缓存
------------------------

你类的一些 metadata 信息解析来自不同的源文件，如 YAML，XML，Annotations 等等。如果不想每个请求都解析这些信息，我们需要使用缓存。

就像查询和结果缓存一样，我们首先需要配置。

.. code-block:: php

    <?php
    $config->setMetadataCacheImpl(new \Doctrine\Common\Cache\ApcCache());

现在 metadata 信息只在第一次解析的时候就存储在缓存中了。

23.4. 清除缓存
------------------

我们已经在前面学些了如何通过缓存 API 去手动的删除缓存数据，为了方便，我们提供了命令行模式帮助你清除 query，result，metadata 缓存。

在 doctrine 的命令行，你可以运行下面的命令：

清除 query 缓存，使用 ``orm:clear-cache:query`` 选项。

.. code-block:: php

    $ ./doctrine orm:clear-cache:query

清除 metadata 缓存，使用 ``orm:clear-cache:metadata`` 选项。

.. code-block:: php

    $ ./doctrine orm:clear-cache:metadata

清除 result 缓存，使用 ``orm:clear-cache:result`` 选项。

.. code-block:: php

    $ ./doctrine orm:clear-cache:result

所有这些命令都接受 ``--flush`` 选项来刷新缓存的整个内容，而不是使缓存无效。

23.5. 缓存链
---------------------

常见的模式是使用静态缓存来存储在单个 PHP 请求中多次请求的数据。即使这些数据可能存储在快速存储器高速缓存中，通常，缓存通过网络链路导致相当大的网络流量。

ChainCache 类允许一次注册多个缓存。例如，如果 ArrayCache 未命中，则可以首先使用每个请求的 ArrayCache，后跟一个（相对较慢）的 MemcacheCache。 ChainCache 自动处理将数据推送到链中更快的缓存，并清除整个堆栈中的数据。

ChainCache 按照它们应该被使用的顺序占用一个简单的 CacheProviders 数组。

.. code-block:: php

    $arrayCache = new \Doctrine\Common\Cache\ArrayCache();
    $memcache = new Memcache();
    $memcache->connect('memcache_host', 11211);
    $chainCache = new \Doctrine\Common\Cache\ChainCache([
        $arrayCache,
        $memcache,
    ]);

ChainCache 本身扩展了 CacheProvider 接口，因此可以创建链条链。虽然这似乎是构建简单高可用性缓存的简单方法，但 ChainCache 不会执行任何异常处理，因此不建议将其用作高可用性机制。

23.6. 缓存碰撞
---------------

需要注意当使用缓存出现缓存碰撞，如果你有一个并发非常高的网站。如果缓存不存在，就会生成缓存信息并保存。现在，如果你的网站有100个请求同时的检查到了缓存不存在并且尝试保存相同的缓存信息，那么将锁住 APC，Xcache 等等，这将会导致一些问题。一个现成的解决方案是通过预先的填充缓存，而不是通过用户请求来填充缓存。

你可以阅读 `这个 <http://notmysock.org/blog/php/user-cache-timebomb.html>`_ 博客获得更多信息。

