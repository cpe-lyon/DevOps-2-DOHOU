# Day 2 - DevOps 2 - Medhy DOHOU - IRC 2025
Lien github : https://github.com/cpe-lyon/DevOps-2-DOHOU/tree/day2
## Déployer l'API

### Que se passe-t-il au niveau des pods de l'API ?

```
 🐈 medhy 08:23:17 12/03/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day2  😸  kubectl logs -f api-75cfdcf679-lqxs8
wait-for-it.sh: waiting 30 seconds for postgres:3306
wait-for-it.sh: timeout occurred after waiting 30 seconds for postgres:3306
```

On remarque que le pod crash car celui-ci attends la connexion à la base de données. Or nous n'avons pas encore monté cette base. Donc le conteneur crash.

## Créer la base de données

## Pointer l'API sur la base de données

### Quel est le nom du service de la base de données ?

Dans mon cas : `database-service`. L'alias DNS interne est donc `database-service.medhy-dohou`

## C'est au tour du front

### Pourquoi plus rien ne fonctionne ? Pourquoi faut-il `kubectl rollout restart deployment MON_API` ?

Plus rien ne fonctionne, car comme les containers sont stateless, c'est à dire sans persistance, le schéma de base de données initialisé au démarrage de l'application à été supprimé, et donc l'api ne peut plus requêter la base.

## La persistance dans Kubernetes

### D’après le tableau, quel est le type d’accès implémenté par notre Storage Class EBS ? Pourquoi cela convient parfaitement pour la persistance de la base de données Postgres ?

Le stockage EBS est en accés ReadWriteOnce; Cela signifie qu'un seul noeud du cluster peut monter ce PVC. Cela est important pour une base, car cela permet de rendre consistent l'écriture sur le disque plutôt que d'avoir plusieurs noeuds qui écrivent de manière concurrente sur le PVC. Par contre, cela signifie que si nous voulons mettre en place un cluster PG, nous devons absolument utiliser un seul noeud du cluster (via l'affinité de noeud et la teinte).