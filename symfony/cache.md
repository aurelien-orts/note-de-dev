Gestion du cache
================

CacheClearerInterface
---------------------

Cette interface permet d'effectuer une action pendant le `cache:clear`

```php
<?php

namespace App\Application\Kernel;

use Symfony\Component\HttpKernel\CacheClearer\CacheClearerInterface;

class CacheClear implements CacheClearerInterface
{
    public function clear($cacheDir)
    {
        //Do everythings
    }
}
```

CacheWarmerInterface
--------------------

Cette interface permet d'effectuer une action pendant le `cache:warmup`

```php
<?php

namespace App\Application\Kernel;

use Symfony\Component\HttpKernel\CacheWarmer\CacheWarmerInterface;

class CacheWarmer implements CacheWarmerInterface
{
    public function warmUp($cacheDir)
    {
        //Do everythings
    }
}
```

Configuration des pools
----------------------

```yaml
parameters:
    redis_dsn: 'redis://redis'    

framework:
    cache:
        default_redis_provider: '%redis_dsn%/2'
        pools:
            opcache.pool:
                adapter: 'cache.adapter.opcache'
            redis.pool:
                adapter: 'cache.adapter.redis'

services:
    cache.adapter.opcache:
        class: Symfony\Component\Cache\Adapter\PhpFilesAdapter
        arguments:
            - ~
            - 0
            - '%kernel.cache_dir%/opcache_files'
        tags:
            - cache.pool
```

**Remarques :** 

Il est parfois nécessaire de versionner le cache pour autoriser une execution de l'application dans 2 versions différentes en meme temps.
Le cas le plus classique, c'est pendant un déploiement sur plusieurs instance

Pour ca, on peut utiliser le namespace:

```yaml
parameters:
    app.cache_version: '1.0.0_'

framework:
    cache:
        prefix_seed: '%app.cache_version%'
```

Attention car le seed est utilisé par l’ensemble des pools, par conséquent l'ensemble des caches seront invalidés
Si les caches sont des optims, alors pas de problème, par contre si les caches sont indispensables, alors il faut penser à les générer avant la mise en ligne 

Doctrine
--------

Doctrine 3 types de cache:
- metadata
- query
- result

Dans l'exemple ci-dessous, nous avons utilisé `opcache` pour optimiser doctrine en env de prod 

```yaml
# config/packages/prod/doctrine.yaml

doctrine:
   orm:
        entity_managers:
            default:
                metadata_cache_driver:
                    type: pool
                    pool: opcache.pool
                query_cache_driver:
                    type: pool
                    pool: opcache.pool
```

Le cache de résultat quand à lui est confié à rédis et est configurer via le bundle `SncRedis` 

```yaml
snc_redis:
  doctrine:
    result_cache:
      client: doctrine
      entity_manager: [default]
      namespace: '%app.cache_version%' #Ici on retrouve le namespace
  clients:
    doctrine: #used by doctrine
      type: predis
      alias: doctrine
      dsn: '%redis_dsn%/1'
```