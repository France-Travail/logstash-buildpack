# Logstash Buildpack

Buildpack Scalingo pour installer Logstash.

Fork du [buildpack officiel Scalingo](https://github.com/Scalingo/logstash-buildpack), apportant deux corrections :

1. **Cohabitation multi-buildpack** : le buildpack officiel utilise `mv` pour installer Logstash dans `$BUILD_DIR`, ce qui échoue si un buildpack
   précédent a déjà créé des répertoires communs (ex : `bin/`) :
   ```
   mv: cannot overwrite '.../bin': Directory not empty
   ```
   Ce fork remplace le `mv` par un `cp -r` qui merge l'arborescence Logstash dans `$BUILD_DIR` sans écraser les fichiers déjà présents,
   permettant la cohabitation avec d'autres buildpacks.

2. **Suppression du plugin `logstash-output-opensearch`** : le buildpack officiel installe ce plugin par défaut, ce qui n'est pas nécessaire
   si la sortie cible est Elasticsearch/Elastic Cloud, et allonge inutilement le build.

## Variables d'environnement

### `LOGSTASH_VERSION`

Version de Logstash à installer. Définie dans `VERSIONS` (défaut : `9.4.5`).

Peut être surchargée via une variable d'environnement Scalingo.

### `LOGSTASH_PLUGINS`

Liste de plugins Logstash supplémentaires à installer au build, séparés par des virgules.

Exemple : `LOGSTASH_PLUGINS=logstash-filter-translate,logstash-output-opensearch`

### `LS_JAVA_OPTS`

Options JVM passées à Logstash.

Heap fixe recommandée sur conteneur XL Scalingo (2 Go) : `LS_JAVA_OPTS="-Xms1g -Xmx1g"`.

> ⚠️ Logstash 9.x affiche un warning si `JAVA_OPTS` est présent dans l'environnement. Utiliser uniquement `LS_JAVA_OPTS`.
> Sur un conteneur XL (2 Go), ne pas dépasser `-Xmx1g` : Logstash consomme ~0,8-1 Go hors heap (Netty, JRuby, metaspace). `-Xmx2g` = OOM-kill.

## Utilisation

Dans le fichier `.buildpacks` de l'application :

```
https://github.com/France-Travail/logstash-buildpack
```

Ce buildpack doit être **en dernier** dans `.buildpacks` : c'est son `bin/release` qui définit le PATH (`/app/bin`) et le process type par défaut.
