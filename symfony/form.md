Les formulaires
===============

Les extensions
--------------

Les extensions de formulaire permettent de centraliser une configuration qui est utilisé par un type de form précis.

Dans le cas ci-dessous, nous avons une extension `PositionTypeExtension` qui sera ajouté à tous les types `IntegerType`

```yaml
services:
    App\Form\Extension\PositionTypeExtension:
        tags:
            - { name: form.type_extension, extended_type: Symfony\Component\Form\Extension\Core\Type\IntegerType }
```

Cette extension permet d'ajouter un attribut html lors du rendu.

```php
<?php

namespace App\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\Extension\Core\Type\IntegerType;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\Form\FormView;

final class PositionTypeExtension extends AbstractTypeExtension
{
    public function buildView(FormView $view, FormInterface $form, array $options)
    {
        $view->vars['attr']['data-type'] = 'position_type';
    }

    public function getExtendedType()
    {
        return IntegerType::class;
    }
}
```

Les Subscribers
---------------

Dans le cas ci-dessous, on utilise l'injection de dépendence initialiser le subscriber.

Puis on ajoute cette instance au builder

```php
<?php

namespace App\Form;

use App\Form\Subscriber\AppSampleTypeSubscriber;
use Symfony\Bridge\Doctrine\Form\Type\TextType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

final class AppSampleType extends AbstractType
{
    private $subscriber;

    public function __construct(AppSampleTypeSubscriber $subscriber)
    {
        $this->subscriber = $subscriber;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name', TextType::class);

        $builder->addEventSubscriber($this->subscriber);
    }
}
```

Validation
----------

Dans l'expemple ci-dessous, les 2 principaux cas de contraintes:
- NotBlank: c'est un contrainte standard comme il en existe bien d'autre dans Symfony
- Callback: elle est très pratique pour effectuer un controle précis. Ex: Si l'API est activé, alors on doit remplir le user/password

```
<?php

namespace App\Form\BackOffice\Collector;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints\NotBlank;

final class AppSampleType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add($name, TextType::class, [
                'constraints' => [
                    new NotBlank(['message' => 'app.form.not_blank']),
                    new Callback([$this, 'validateJson']),
                ],
            ]);
    }

    public function validateJson(?string $value, ExecutionContextInterface $context): void
    {
        $form = $context->getRoot();
        $entity = $form->getData();

        if (!$entity instanceof AppEntity) {
            return;
        }

        if (false /* Remplacer par un vrai test */) {
            $context->addViolation(
                $this->translator->trans('app.form.validate_json')
            );
        }
    }
}
```