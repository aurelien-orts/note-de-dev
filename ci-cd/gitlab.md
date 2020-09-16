Gitlab
======

Dans l'exemple ci-dessous présente une configuration CI/CD pour un projet Symfony sur un environement AWS

````yaml
# gitlab-ci.yml
image: aorts/ci:latest

services:
    - postgres:10.4-alpine
    - redis:5.0.5-alpine

stages:
    - Build
    - Code Quality
    - Tests
    - Deploy

variables: &variables
    AWS_DEFAULT_REGION: eu-west-1

build:
    stage: Build
    artifacts:
        name: $CI_COMMIT_REF_NAME
        paths:
            - /builds/nom-du-repos
    cache:
        paths:
            - node_modules/
            - vendor/
    script:
        - "cp config/parameters.yml.dist config/parameters.yml"
        - sh deploy/prepare_gitinfo.bash
        - composer install --no-scripts --no-progress
        - bin/console doctrine:migrations:migrate --no-interaction
        - bin/console doctrine:schema:validate
        - bin/console cache:clear --no-warmup
        - bin/console doctrine:fixtures:load --group=prod -n
        - yarn install
        - bin/console assets:install web
        - bin/console fos:js-routing:dump --format=json --target=assets/backoffice/dist/fos_js_routes.json
        - bin/console bazinga:js-translation:dump web/extra
        - yarn encore prod

phpcs:
    stage: Code Quality
    except:
        - development
        - master
    script:
        - vendor/bin/phpcs -s
    dependencies:
        - build

phpunit:
    stage: Tests
    before_script:
        - bin/console doctrine:migrations:migrate
    script:
        - vendor/bin/phpunit --configuration phpunit.xml.dist
    dependencies:
        - build

deploy_release:
    image: garland/aws-cli-docker
    stage: Deploy
    dependencies:
        - build
    environment:
        name: release
    variables:
        <<: *variables
        S3_BUCKET_NAME: your-bucket
        DEPLOYMENT_GROUP_NAME: release
    artifacts:
        paths:
            - /builds/nom-du-repos/push_output.sh
        when: on_failure
    before_script:
        - rm -rf var/logs/**
        - rm -rf var/cache/**
        - find . -mtime +10950 -exec touch {} \; #permet de corriger un bug sur les datetimes présent dans les vendors
    script:
        - aws deploy push --application-name $CD_APP_NAME --s3-location s3://$S3_BUCKET_NAME/deployment/app.zip --source . > push_output.sh
        - sed -i 's/To deploy with this revision, run://g' push_output.sh
        - sed -i 's/<deployment-group-name>/release/g' push_output.sh
        - sed -i 's/<deployment-config-name>/CodeDeployDefault.OneAtATime/g' push_output.sh
        - sed -i "s/<description>/$CI_COMMIT_SHA/g" push_output.sh
        - sed -i 's/aws //g' push_output.sh
        - aws $(cat ./push_output.sh)
    when: on_success
    only:
        - /^release$/
````

Authentification SSH sur des depots privées
-------------------------------------------

Lorsqu'on ajoute une dépendance composer, il est possible que le dépot soit privée.

Pour que la CI s'authentifie, la seule solution à date proposée par Gitlab est de créer une clé SSH en la passant par les variables du projet

Dans le cas ci-dessous, la variable SSH_PRIVATE_KEY contient le contenu d'une clé SSH privé

Cette configuration est possible dans "Settings/CI / CD/Variables":
![Variables](gitlab-variables.png)

````yaml
build:
    before_script:
        - 'which ssh-agent || ( apt-get install -qq openssh-client )'
        - eval $(ssh-agent -s)
        - ssh-add <(echo "$SSH_PRIVATE_KEY")
        - mkdir -p ~/.ssh
        - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
````