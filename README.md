
# Recette de génération des images Docker Apache

Ce dépôt contient les recettes permettant de générer les images Apache du Gouvernement de la Polynésie Française.

Fonctionnalités :
- Exécution en non-root user (uniquement la version **2.4-bullseye**)
- Possibilité d'exécuter des scripts au démarrage du container avant le lancement de Apache
- Reconfiguration du port sur lequel Apache écoute via variable d'environnement
- Configuré avec **ServerTokens Prod** et **ServerSignature Off**
- Configuration par défaut de quelques entêtes : X-Frame-Options, CSP, STS...

**Warning!!!** Les versions buster et stretch sont dépréciées. Veuillez passer sur la **bullseye**. Afin d'éviter tout éventuel problème suite aux changements effectués dans **bullseye**, **govpf/apache:2.4** continuera à pointer sur **buster** _**avant**_ la suppression des version **buster** et **stretch**.

# How to use

## Create a `Dockerfile`

```dockerfile
FROM govpf/apache:2.4
COPY ./dist /var/www/html
# Optional, scripts will be run right before starting apache2 service
COPY ./scripts/*.sh /docker-entrypoint-init.d/
```

### With `docker run`

```bash
# Bullseye's tag (run as apache user)
docker run --name web -dit -p 8080:8080 -v $PWD:/var/www/html govpf/apache:2.4-bullseye

# Stretch's tag (run as root)
docker run --name web -dit -p 8080:80 -v $PWD:/var/www/html govpf/apache:2.4-stretch
docker run --name web -dit -p 8080:80 -v $PWD:/var/www/html govpf/apache:2.4

# Buster tag (run as root)
docker run --name web -dit -p 8080:80 -v $PWD:/var/www/html govpf/apache:2.4-buster
```

# Configuration

## Changer le port de Apache

```Dockerfile
FROM govpf/apache:2.4
ENV APACHE_HTTP_PORT 9000
```
**Attention**
A partir de 2.4-bullseye, il n'est plus possible de mettre un port inférieur à 1024 à cause de la restriction de privilèges. La `2.4` continuera à pointer sur `2.4-buster` jusqu'à la suppresion des versions `buster` et `stretch`

## Configuration du site

```Dockerfile
FROM govpf/apache:2.4
COPY ./000-default.conf /etc/apache2/sites-available/000-default.conf
```

## Ajouter des scripts à exécuter au démarrage du container

Les scripts dans le répertoire `/docker-entrypoint-init.d/` sont exécutés séquentiellement par ordre ASC.

### Exemple

001-init.sh
```bash
#!/bin/bash
sed -i 's/something/hello/g' /var/www/html/main.js
....
```
Dockerfile
```Dockerfile

FROM govpf/apache:2.4
COPY ./scripts/001-init.sh /docker-entrypoint-init.d/
```
## Spécificités des variantes

### Non-root user

Seul le tag `2.4-bullseye` est en `non-root user` actuellement. Le container est lancé en tant que `apache`. L'utilisateur `apache` fait parti du group `www-data` et a les droits `rw` sur `/etc/apache2` et `/var/www/html`.

Les versions `2.4`, `2.4-stretch` et `2.4-buster` utilisent le user `root` pour pouvoir écouter sur le port `80` par défaut.

Dans les deux cas, le process apache est exécuté en tant que `www-data`.

### Port d'écoute par défaut

Seul le tag `2.4-bullseye` écoute sur le port `8080` par défaut.

Les versions `2.4`, `2.4-stretch` et `2.4-buster` écoutent le port `80` par défaut.

### Entêtes de sécurité

Les versions `2.4`, `2.4-stretch` et `2.4-buster`, par défaut, n'ont que les entêtes **ServerTokens Prod** et **ServerSignature Off** de configurés.
