YAML
====

Valider un yaml
---------------

Pour plus de détail, voir la [documentation](https://symfony.com/doc/current/components/config/definition.html)

```php
<?php

namespace App\Service\Reader;

use App\DependencyInjection\Configuration\YourYamlConfiguration;
use Symfony\Component\Config\Definition\Processor;
use Symfony\Component\Yaml\Yaml;

class ReaderService
{
    public function validate() {
        $yamlData = Yaml::parseFile('__DIR__./your-file.yaml');
        $processor = new Processor();
        $configuration = new YourYamlConfiguration(); //Read the doc
        $config = $processor->processConfiguration($configuration, $yamlData);
    }        
}
```