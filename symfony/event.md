Gestion des evenements
======================

Event
-----

Quand on dispatch un événement, il est préférable d'envoyer l'object à l'origine de cette événement.

Pour ce faire, il faut créer une classe qui étends `Event` avec un constructeur et un setter de l'objet emmeteur.

Dans notre cas, on considère que l'object emetteur est un `MyEntity`

```php
<?php

namespace App\Event;

use App\Entity\MyEntity;
use Symfony\Component\EventDispatcher\Event;

class MyEntityEvent extends Event
{
    private $myEntity;

    public function __construct(MyEntity $myEntity)
    {
        $this->myEntity = $myEntity;
    }

    public function getMyEntity(): MyEntity
    {
        return $this->myEntity;
    }
}
```

Dispatcher
----------

Le dispatcher est le service permettant de lancer un evenement

```php
<?php

namespace App\Service;

use App\AppEvents;
use App\Event\MyEntityEvent;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class MyEntityImporter
{
    /** @var EventDispatcherInterface $dispatcher */
    private $dispatcher;

    public function __construct(EventDispatcherInterface $dispatcher) {
        $this->dispatcher = $dispatcher;
    }


    public function import()
    {
        $myEntityEvent = new MyEntityEvent($myEntity);
        $this->dispatcher->dispatch(AppEvents::BEFORE_IMPORT, $myEntityEvent);
        //Make treatment ...
        $this->dispatcher->dispatch(AppEvents::AFTER_IMPORT, $myEntityEvent);
    }
}
```

Subscriber
----------

Le subscriber ecoute un ou plusieurs evenements

```php
<?php

namespace App\EventSubscriber;

use App\EopEvents;
use App\Event\MyEntityEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

class MyEntitySubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            EopEvents::BEFORE_IMPORT => 'beforeImport',
            EopEvents::AFTER_IMPORT => 'afterImport',
        ];
    }

    public function beforeImport(MyEntityEvent $myEntityEvent): void
    {
        //Do Everythings
    }

    public function afterImport(MyEntityEvent $myEntityEvent): void
    {
        //Do Everythings
    }
}
```

Listing
-------

Pour simplifier la recherche, j'opte pour la centralisation dans une classe de tous les évenements de l'application

Pour ce faire, je crée une classe à la racine du projet dans `src`

On peut lister tous les events de l'application avec leurs indices de priorité via la commande: `php bin/console debug:event`

```php
<?php

namespace App;

final class EopEvents
{
    public const BEFORE_IMPORT = 'app.import.before';
    public const AFTER_IMPORT = 'app.import.after';
}
```