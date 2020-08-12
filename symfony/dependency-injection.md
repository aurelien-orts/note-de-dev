Injection de dependence
=======================

Tag automatique de classe
-------------------------

Toutes les classes qui implémenteront l'interface `TagInterface` seront taggées __app.tag__

```yaml
services:
    _instanceof:
        App\Service\TagInterface:
            tags: ['app.tag']

```

Tag automatique par dossier
---------------------------

Toutes les classes contenu dans le dossier spécifier seront taggées __doctrine.event_subscriber__

```yaml
services:
    App\Subscriber\Doctrine\:
        resource: '../src/Subscriber/Doctrine'
        tags: ['doctrine.event_subscriber']
```


Binder les paramètres
---------------------

Ci-dessous, les différentes manières de binder des variables

```yaml
services:
    _defaults:
        bind:
            $analyticsType: '%app.analytics.type%'        #bind d'un paramètre simple
            $paymentWorkflow: '@workflow.payment_status'  #bind service simple
            $tagList: !tagged app.tag                     #bind de service en utilisant les tags
```

Services et Alias
-----------------

```yaml
services:
    #Create an alias
    Symfony\Component\Templating\TemplateNameParserInterface: '@templating.name_parser'

    #Reference new service. This name will be set with her FQCN
    Symfony\Component\String\Slugger\AsciiSlugger:
```