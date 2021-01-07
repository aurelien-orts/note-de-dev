Composer
========

Composer access token
---------------------

Configurer la connexion par token de composer

[Source](https://www.previousnext.com.au/blog/managing-composer-github-access-personal-access-tokens)

Initialiser un nouveau token dans github en autorisant tous les Repo puis exécuter le commande:

```bash
composer config -g github-oauth.github.com XXXXXXXXXXXXXXXXXXXXXXX
```