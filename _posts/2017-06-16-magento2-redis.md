---
date: 2017-06-16
title: Use Redis for Magento2 cache and session
categories:
  - Tutorial
description: "Use Redis for Magento2 cache and session"
type: Document
---

## Install Redis
We use Ubuntu 16.04 as our main os, we will install redis now.

Prerequisites:
```
$ sudo apt-get update
$ sudo apt-get install build-essential
$ sudo apt-get install tcl8.5
```
Installing Redis
```
$ wget http://download.redis.io/releases/redis-stable.tar.gz
$ tar xzf redis-stable.tar.gz
$ cd redis-stable
```    
Proceed to with the make command
```   
$ make
$ make test
$ sudo make install
```
Once the program has been installed, Redis comes with a built in script that sets up Redis to run as a background daemon. 

To access the script move into the utils directory:
   
    cd utils
    sudo ./install-server.sh
   
As the script runs, you can choose the default options by pressing enter. Once the script completes, the redis-server will be running in the background.
   
You can start and stop redis with these commands (the number depends on the port you set during the installation. 6379 is the default port setting):
```
$ sudo service redis_6379 start
$ sudo service redis_6379 stop      
```    

## Securing Redis

By default, Redis server allows connections from anywhere which is insecure. Binding to localhost will restrict access to the server itself and is a good first step to protecting your server.

Open the Redis configuration file for editing:

    sudo vi /etc/redis/6379.conf

Locate this line and make sure it is uncommented (remove the `#` if it exists):

    bind 127.0.0.1

This is only a first step to securing your Redis instance, You may need more actions to protect your server.
   
## Test the Redis Instance Functionality

To test that your service is functioning correctly, input:

    $ redis-cli ping

You should see:
    
    PONG

   
## Configure Magento to use Redis for default and page caching
Following is a sample configuration that causes Magento to use Redis for both the default cache (default array) and the full page cache (page_cache array).

Add a configuration similar to the following to <your Magento install dir>app/etc/env.php:
    
    'cache' =>
    array(
       'frontend' =>
       array(
          'default' =>
          array(
             'backend' => 'Cm_Cache_Backend_Redis',
             'backend_options' =>
             array(
                'server' => '127.0.0.1',
                'database' => '0',
                'port' => '6379'
                ),
        ),
        'page_cache' =>
        array(
          'backend' => 'Cm_Cache_Backend_Redis',
          'backend_options' =>
           array(
             'server' => '127.0.0.1',
             'port' => '6379',
             'database' => '1',
             'compress_data' => '0'
           )
        )
      )
    ),

    
## Configure Magento to use Redis for session storage  
Open <your Magento install dir>app/etc/env.php, it may has session configure before, you can find below lines in env.php

      'session' => 
      array (
        'save' => 'files',
      ),
      
Replace them by below configuration:
      
    'session' => 
       array (
       'save' => 'redis',
       'redis' => 
          array (
    	'host' => '127.0.0.1',
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
    	'max_lifetime' => '2592000'
        )
    ),
      
     
## Redis monitor command
    
In a command prompt on the server on which Redis is running, enter:
    
    $ redis-cli monitor
    
Refresh your storefront page and you’ll see output similar to the following.
   
If you use Redis for session storage, you’ll see output similar to the following:

``` 
1476824834.187250 [0 127.0.0.1:52353] "select" "0"
1476824834.187587 [0 127.0.0.1:52353] "hmget" "sess_sgmeh2k3t7obl2tsot3h2ss0p1" "data" "writes"
1476824834.187939 [0 127.0.0.1:52353] "expire" "sess_sgmeh2k3t7obl2tsot3h2ss0p1" "1200"
1476824834.257226 [0 127.0.0.1:52353] "select" "0"
1476824834.257239 [0 127.0.0.1:52353] "hmset" "sess_sgmeh2k3t7obl2tsot3h2ss0p1" "data" "_session_validator_data|a:4:{s:11:\"remote_addr\";s:12:\"10.235.34.14\";s:8:\"http_via\";s:0:\"\";s:20:\"http_x_forwarded_for\";s:0:\"\";s:15:\"http_user_agent\";s:115:\"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36\";}_session_hosts|a:1:{s:12:\"10.235.32.10\";b:1;}admin|a:0:{}default|a:2:{s:9:\"_form_key\";s:16:\"e331ugBN7vRjGMgk\";s:12:\"visitor_data\";a:3:{s:13:\"last_visit_at\";s:19:\"2016-10-18 21:06:37\";s:10:\"session_id\";s:26:\"sgmeh2k3t7obl2tsot3h2ss0p1\";s:10:\"visitor_id\";s:1:\"9\";}}adminhtml|a:0:{}customer_base|a:1:{s:20:\"customer_segment_ids\";a:1:{i:1;a:0:{}}}checkout|a:0:{}" "lock" "0"

... more ...
```

If you use Redis for page caching, you’ll see output similar to the following:

    1476826133.810090 [0 127.0.0.1:52366] "select" "1"
    1476826133.816293 [0 127.0.0.1:52367] "select" "0"
    1476826133.817461 [0 127.0.0.1:52367] "hget" "zc:k:ea6_GLOBAL__DICONFIG" "d"
    1476826133.829666 [0 127.0.0.1:52367] "hget" "zc:k:ea6_DICONFIG049005964B465901F774DB9751971818" "d"
    1476826133.837854 [0 127.0.0.1:52367] "hget" "zc:k:ea6_INTERCEPTION" "d"
    1476826133.868374 [0 127.0.0.1:52368] "select" "1"
    1476826133.869011 [0 127.0.0.1:52369] "select" "0"
    1476826133.869601 [0 127.0.0.1:52369] "hget" "zc:k:ea6_DEFAULT_CONFIG_CACHE_DEFAULT__10__235__32__1080MAGENTO2" "d"
    1476826133.872317 [0 127.0.0.1:52369] "hget" "zc:k:ea6_INITIAL_CONFIG" "d"
    1476826133.879267 [0 127.0.0.1:52369] "hget" "zc:k:ea6_GLOBAL_PRIMARY_PLUGIN_LIST" "d"
    1476826133.883312 [0 127.0.0.1:52369] "hget" "zc:k:ea6_GLOBAL__EVENT_CONFIG_CACHE" "d"
    1476826133.898431 [0 127.0.0.1:52369] "hget" "zc:k:ea6_DB_PDO_MYSQL_DDL_STAGING_UPDATE_1" "d"
    1476826133.898794 [0 127.0.0.1:52369] "hget" "zc:k:ea6_RESOLVED_STORES_D1BEFA03C79CA0B84ECC488DEA96BC68" "d"
    1476826133.905738 [0 127.0.0.1:52369] "hget" "zc:k:ea6_DEFAULT_CONFIG_CACHE_STORE_DEFAULT_10__235__32__1080MAGENTO2" "d"
    
    ... more ...

## Notice
If the redis server is down, the Magneto2 site will crash also, to test it, stop redis server:
    
    sudo service redis_6379 stop
    
Then refresh Magento2 site, you will got below error message:
    
    CredisException: Connection to Redis failed after 2 failures.Last Error : (111) Connection refused in /var/www/html/magento2/vendor/colinmollenhour/credis/Client.php:448 Stack trace: #0 /var/www/html/magento2/vendor/colinmollenhour/credis/Client.php(444): Credis_Client->connect() #1 /var/www/html/magento2/vendor/colinmollenhour/credis/Client.php(774): Credis_Client->connect() #2 /var/www/html/magento2/vendor/colinmollenhour/credis/Client.php(604): Credis_Client->__call('select', Array) #3 /var/www/html/magento2/vendor/colinmollenhour/cache-backend-redis/Cm/Cache/Backend/Redis.php(144): Credis_Client->select(1) #4 /var/www/html/magento2/vendor/magento/zendframework1/library/Zend/Cache.php(153): Cm_Cache_Backend_Redis->__construct(Array) #5 /var/www/html/magento2/vendor/magento/zendframework1/library/Zend/Cache.php(94): Zend_Cache::_makeBackend('Cm_Cache_Backen...', Array, true, true) #6 /var/www/html/magento2/vendor/magento/framework/App/Cache/Frontend/Factory.php(155): Zend_Cache::factory('Magento\\Framewo...', 'Cm_Cache_Backen...', Array, Array, true, true, true) #7 /var/www/html/magento2/vendor/magento/framework/App/Cache/Frontend/Pool.php(67): Magento\Framework\App\Cache\Frontend\Factory->create(Array) #8 /var/www/html/magento2/vendor/magento/framework/App/Cache/Frontend/Pool.php(146): Magento\Framework\App\Cache\Frontend\Pool->_initialize() #9 /var/www/html/magento2/vendor/magento/framework/App/Cache/Type/FrontendPool.php(84): Magento\Framework\App\Cache\Frontend\Pool->get('default') #10 /var/www/html/magento2/vendor/magento/framework/App/Cache/Type/Config.php(49): Magento\Framework\App\Cache\Type\FrontendPool->get('config') #11 /var/www/html/magento2/vendor/magento/framework/Cache/Frontend/Decorator/Bare.php(65): Magento\Framework\App\Cache\Type\Config->_getFrontend() #12 /var/www/html/magento2/vendor/magento/framework/App/ObjectManager/ConfigLoader.php(66): Magento\Framework\Cache\Frontend\Decorator\Bare->load('global::DiConfi...') #13 /var/www/html/magento2/vendor/magento/framework/App/ObjectManager/Environment/Developer.php(77): Magento\Framework\App\ObjectManager\ConfigLoader->load('global') #14 /var/www/html/magento2/vendor/magento/framework/App/ObjectManagerFactory.php(194): Magento\Framework\App\ObjectManager\Environment\Developer->configureObjectManager(Object(Magento\Framework\Interception\ObjectManager\Config\Developer), Array) #15 /var/www/html/magento2/vendor/magento/framework/App/Bootstrap.php(385): Magento\Framework\App\ObjectManagerFactory->create(Array) #16 /var/www/html/magento2/vendor/magento/framework/App/Bootstrap.php(232): Magento\Framework\App\Bootstrap->initObjectManager() #17 /var/www/html/magento2/index.php(38): Magento\Framework\App\Bootstrap->createApplication('Magento\\Framewo...') #18 {main}

Enter below command to start redis server:
    
    sudo servcie redis_6379 start
    
Refresh Magento2 site, it restore normal.    
    
## Reference

* Magento DevDocs [DevDocs](http://devdocs.magento.com/guides/v2.0/config-guide/redis/config-redis.html)
* Redis Official Site [redis.io](https://redis.io/topics/quickstart)
* DigitalOcean [redis installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-redis)
