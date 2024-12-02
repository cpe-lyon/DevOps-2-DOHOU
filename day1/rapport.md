# DevOps 2 - Medhy DOHOU - IRC 2025
Lien github : https://github.com/cpe-lyon/DevOps-2-DOHOU/tree/day1
## Jour 1

### Quelles sont les informations que l'on retrouve dans ce fichier ?

On retrouve dans ce fichier de configuration plusieurs informations :

- `kind: Config` : le type d'objet que décrit ce fichier, à savoir, une configuration
- `users: ...` : une liste des utilisateurs qui accéderont aux clusters kube décris dans la partie `clusters`; On y retrouve leur nom d'utilisateurs, mais aussi leur token de connexion
- `contexts: ...` : une liste des contextes liés aux différents clusters décrit dans le fichier. Ici on y retrouve l'utilisateur à utiliser lors de la connexion à un cluster en particulier
- `clusters: ...` : une liste de cluster kubernetes distant avec l'hostname du cluster; le certificat du cluster distant est également décrit ici

### Quelle est la différence ?

```
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:tech:medhy-dohou" cannot list resource "pods" in API group "" in the namespace "default"
```

### Etape 1
#### Quelles sont les propriétés principales que l'on retrouve dans le fichier yaml de description du pod ?

On y retrouve sous la clé `spec` les spécification de nos containers, sous forme de liste :
- `image` : décrit l'image qui est utilisé par le container
- `imagePullPolicy` : permet de définir la politique de mise à jour de l'image, ici elle est à `Always`
- `name`: le nom du container qui sera instancié
- `resources` avec les `limits` et les `requests` : ce sont les ressources physiques qui sont demandées par le pod, en termes de CPU et RAM. Le champ `requests` décrit la requête initiale, et le champ `limits` décrit les limites maximales du Pod
- `volumeMounts` : décrit les volume montés sur le container du pod
- `nodeName` : le nom du noeud du cluster qui héberge le pod

#### Que se passe-t-il lors de cette deuxième création en impératif ?

On obtient une erreur `pods "mynginx" already exists`.

#### Que se passe-t-il lors de cette deuxième création en déclaratif ?

Le serveur nous rappelle que nous devons rajouté l'annotation `last-applied-configuration`, qui permet de savoir la dernière date de mise à jour de la ressource distante. Cela permettra notamment de s'assurer que l'on mets à jour la ressource et éviter les conflits de màj.

Cependant, le fichier déclaratif est bien appliqué, puisque kubernetes fait le delta entre la situation actuelle et la déclaration, puis applique les modifications nécessaires.

#### Que remarquez-vous dans la description des properties `spec: template` ?

On ne précise pas les adresses IP, ni la stratégie DNS.

#### À quoi sert le ``selector: matchLabels`` ?

Cela permet d'aggréger les ressources déjà existantes dans le replicaset, mais également d'identifier les ressources qui feront parti du replicaset.

#### Combien y a-t-il de pods déployés dans votre namespace ?

Il y a désormais 3 pods dans le namespace.

#### Supprimez un pod. Que se passe-t-il ?

```shell
🐈 medhy 11:34:09 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl delete pods unicorn-front-replicaset-2n6gx && kubectl get pods -n medhy-dohou
pod "unicorn-front-replicaset-2n6gx" deleted
NAME                             READY   STATUS              RESTARTS   AGE
unicorn-front-replicaset-gh9rb   1/1     Running             0          98s
unicorn-front-replicaset-p8lg8   1/1     Running             0          98s
unicorn-front-replicaset-rf5t6   0/1     ContainerCreating   0          1s
```

Le pod est supprimé, mais directement recréée pour pouvoir satisfaire les demandes de replicas du replicaset.

#### Supprimez le replicaset. Que se passe-t-il ?

```
🐈 medhy 11:35:03 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl delete replicaset unicorn-front-replicaset
replicaset.apps "unicorn-front-replicaset" deleted
🐈 medhy 11:36:48 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl get pods -n medhy-dohou
No resources found in medhy-dohou namespace.
```

Les pods associés sont bien supprimés.

#### Quels sont les changements par rapport au ReplicaSet ?

On remarque l'exposition du port 80 pour le Deployment, probablement afin de pouvoir conduire des tests de vie.

#### Combien de pods sont déployés, combien de replicaset ? 

```
🐈 medhy 12:22:10 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-588848978b-249xx   1/1     Running   0          5s
pod/unicorn-front-deployment-588848978b-8h9t7   1/1     Running   0          5s
pod/unicorn-front-deployment-588848978b-b9fl2   1/1     Running   0          5s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   3/3     3            3           5s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   3         3         3       5s
```

On voit bien 3 pods déployés, et un replicaset.

#### Une fois terminé, combien y a-t-il de replicaset ? Combien y a-t-il de Pods ? 

```
🐈 medhy 12:24:39 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-77fcc4b7bd-cw7vk   1/1     Running   0          2m46s
pod/unicorn-front-deployment-77fcc4b7bd-k9xl6   1/1     Running   0          2m49s
pod/unicorn-front-deployment-77fcc4b7bd-qxk2c   1/1     Running   0          2m48s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   3/3     3            3           5m11s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   0         0         0       5m11s
replicaset.apps/unicorn-front-deployment-77fcc4b7bd   3         3         3       2m49s
```

L'ancien replicaset à été conservé pour pouvoir eventuellement être rollbacké. On peut cependant voir qu'il est inactif (0 replicas, 0 actifs, 0 désiré). On retrouve bien de nouveau 3 pods associés au nouveau déploiement.

#### Allez voir les logs des événements du déploiement avec kubectl describe deployments. Qu’observez-vous ?

```
🐈 medhy 12:24:31 12/02/24   main ?:1  🏚️/git/DevOps-2-DOHOU/day1  😸  kubectl describe deployments
Name:                   unicorn-front-deployment
Namespace:              medhy-dohou
CreationTimestamp:      Mon, 02 Dec 2024 12:22:09 +0100
Labels:                 app=unicorn-front
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=unicorn-front
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
Labels:  app=unicorn-front
Containers:
unicorn-front:
Image:         registry.takima.io/school/proxy/nginx:1.9.1
Port:          80/TCP
Host Port:     0/TCP
Environment:   <none>
Mounts:        <none>
Volumes:         <none>
Node-Selectors:  <none>
Tolerations:     <none>
Conditions:
Type           Status  Reason
----           ------  ------
Available      True    MinimumReplicasAvailable
Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  unicorn-front-deployment-588848978b (0/0 replicas created)
NewReplicaSet:   unicorn-front-deployment-77fcc4b7bd (3/3 replicas created)
Events:
Type    Reason             Age    From                   Message
----    ------             ----   ----                   -------
Normal  ScalingReplicaSet  2m30s  deployment-controller  Scaled up replica set unicorn-front-deployment-588848978b to 3
Normal  ScalingReplicaSet  8s     deployment-controller  Scaled up replica set unicorn-front-deployment-77fcc4b7bd to 1
Normal  ScalingReplicaSet  7s     deployment-controller  Scaled down replica set unicorn-front-deployment-588848978b to 2 from 3
Normal  ScalingReplicaSet  7s     deployment-controller  Scaled up replica set unicorn-front-deployment-77fcc4b7bd to 2 from 1
Normal  ScalingReplicaSet  5s     deployment-controller  Scaled down replica set unicorn-front-deployment-588848978b to 1 from 2
Normal  ScalingReplicaSet  5s     deployment-controller  Scaled up replica set unicorn-front-deployment-77fcc4b7bd to 3 from 2
Normal  ScalingReplicaSet  4s     deployment-controller  Scaled down replica set unicorn-front-deployment-588848978b to 0 from 1
```

On observe bien que le déploiement à été mis à jour en mode rolling, notamment en passant par le scale down progressif de l'ancien déploiement (passage de 3 à 2, de 2 à 1, puis de 1 à 0 des replicas), et du scale up progressif du nouveau déploiement (la même chose mais dans l'autre sens).

#### Update du déploiement avec une image qui n'existe pas : que se passe-t-il ?

```
NAME                                            READY   STATUS             RESTARTS   AGE
pod/unicorn-front-deployment-77fcc4b7bd-cw7vk   1/1     Running            0          6m26s
pod/unicorn-front-deployment-77fcc4b7bd-k9xl6   1/1     Running            0          6m29s
pod/unicorn-front-deployment-77fcc4b7bd-qxk2c   1/1     Running            0          6m28s
pod/unicorn-front-deployment-84f96bbc4f-6526t   0/1     ImagePullBackOff   0          12s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   3/3     1            3           8m51s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   0         0         0       8m51s
replicaset.apps/unicorn-front-deployment-77fcc4b7bd   3         3         3       6m29s
replicaset.apps/unicorn-front-deployment-84f96bbc4f   1         1         0       12s
```

On remarque que le nouveau déploiement est en erreur `ImagePullBackOff`. Cependant, le fonctionnement de notre applicatif n'est pas impacté : les pods de l'ancien déploiement sont toujours en vie et le restent tant que le nouveau déploiement n'est pas dans un état sain, garantissant une continuité de service dans ce genre de cas. Cela est le resultat de la stratégie RollingUpdate.

#### Combien y'a-t-il de révisions pour ce déploiement, à quoi correspond "ChangeCause" ?

```
 🐈 medhy 12:34:08 12/02/24  🏚️  😸 kubectl rollout history deployment.v1.apps/unicorn-front-deploymentt
deployment.apps/unicorn-front-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment.v1.apps/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.91-falseimage --record=true
```

On observe 3 versions pour ce déploiement. La CHANGE-CAUSE semble correpondre à la commande qui à été lancée pour mettre à jour la ressource. On remarque que lorsque la ressource est mise à jour par le biais déclaratif, la change-cause est vide.

#### Mise à l'échelle : Combien y'a-t-il de pods ?

```
 🐈 medhy 12:36:10 12/02/24  🏚️  😸  kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-77fcc4b7bd-5nwkh   1/1     Running   0          6s
pod/unicorn-front-deployment-77fcc4b7bd-cw7vk   1/1     Running   0          11m
pod/unicorn-front-deployment-77fcc4b7bd-k9xl6   1/1     Running   0          11m
pod/unicorn-front-deployment-77fcc4b7bd-qxk2c   1/1     Running   0          11m
pod/unicorn-front-deployment-77fcc4b7bd-t8jf7   1/1     Running   0          5s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     5            5           14m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   0         0         0       14m
replicaset.apps/unicorn-front-deployment-77fcc4b7bd   5         5         5       11m
replicaset.apps/unicorn-front-deployment-84f96bbc4f   0         0         0       5m28s
```

L'ordre de scale up est bien pris en compte, on a bien 5 pods.

#### Stand-by déploiement: Que se passe-t-il au niveau du replicaset ?

```
 🐈 medhy 12:36:17 12/02/24  🏚️  😸 kubectl rollout pause deployment.v1.apps/unicorn-front-deploymentt
kubectl set image deployment.v1.apps/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.20.2
deployment.apps/unicorn-front-deployment paused
deployment.apps/unicorn-front-deployment image updated
 🐈 medhy 12:37:26 12/02/24  🏚️  😸  kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-77fcc4b7bd-5nwkh   1/1     Running   0          83s
pod/unicorn-front-deployment-77fcc4b7bd-cw7vk   1/1     Running   0          12m
pod/unicorn-front-deployment-77fcc4b7bd-k9xl6   1/1     Running   0          13m
pod/unicorn-front-deployment-77fcc4b7bd-qxk2c   1/1     Running   0          13m
pod/unicorn-front-deployment-77fcc4b7bd-t8jf7   1/1     Running   0          82s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     0            5           15m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   0         0         0       15m
replicaset.apps/unicorn-front-deployment-77fcc4b7bd   5         5         5       13m
replicaset.apps/unicorn-front-deployment-84f96bbc4f   0         0         0       6m44s
```

On remarque que le nouveau replicaset n'est pas mis à jour, et les pods non plus (si l'on regarde leur AGE).

Après execution de la commande resume : 
```
 🐈 medhy 12:37:33 12/02/24  🏚️  😸 kubectl rollout resume deployment.v1.apps/unicorn-front-deploymentt
deployment.apps/unicorn-front-deployment resumed
 🐈 medhy 12:38:41 12/02/24  🏚️  😸  kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-58c88ffb7b-45cg5   1/1     Running   0          18s
pod/unicorn-front-deployment-58c88ffb7b-bfdqn   1/1     Running   0          17s
pod/unicorn-front-deployment-58c88ffb7b-drv82   1/1     Running   0          18s
pod/unicorn-front-deployment-58c88ffb7b-hmblr   1/1     Running   0          17s
pod/unicorn-front-deployment-58c88ffb7b-sk5ck   1/1     Running   0          18s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     5            5           16m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-588848978b   0         0         0       16m
replicaset.apps/unicorn-front-deployment-58c88ffb7b   5         5         5       18s
replicaset.apps/unicorn-front-deployment-77fcc4b7bd   0         0         0       14m
replicaset.apps/unicorn-front-deployment-84f96bbc4f   0         0         0       8m11s
```

Les pods sont bien mis à jour, un nouveau déploiement est crée.

### Etape 2 

#### Mise en situation : Que se passe-t-il ? Pourquoi ?

```
🐈 medhy 14:02:22 12/02/24  🏚️  😸  kubectlget all
NAME                         READY   STATUS             RESTARTS   AGE
pod/hello-79d6647d94-h7wtd   0/1     ImagePullBackOff   0          41s
pod/hello-79d6647d94-sgmcd   0/1     ErrImagePull       0          41s
pod/hello-79d6647d94-zx2p9   0/1     ErrImagePull       0          41s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello   0/3     3            0           41s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-79d6647d94   3         3         0       41s
```

On a bien crée un déploiement qui a créer un replicaset qui à lui même crée 3 pods différents. Cependant ces 3 pods ne peuvent démarrer car ils n'ont pas de credentials pour le registre gitlab.

#### Décrivez ce que répond la Web App ? Actualisez votre page avec CTRL + F5. Que se passe-t-il ?

On remarque que l'on navigue entre 3 pages de couleurs différentes des lors que nous rafraichissons avec CTRL+F5

#### Que constatez-vous sur le navigateur ?

On remarque que l'on a une sticky session avec un des 3 pods tant que l'on ne fait pas un refresh avec vidage du cache (CTRL+F5). Cela est confirmé par l'adresse du noeud qui change entre chaque refresh.

#### Bonus 3 : Observez ce qu'il se passe. Que constatez-vous ?

On observe que dès lors que l'on passe la barre des 50%, l'HPA démarre un nouveau pod afin de tenir la charge et de s'assurer que la charge individuelle CPU d'un Pod ne dépasse pas 50%. Dans notre test, le nombre de pods se stabilise à 6.

#### Bonus 4 : Que constatez-vous ?

On constate un timeout au niveau du WGET : la nouvelle network policy bloque le trafic qui ne provient pas de pod avec le label access=true

#### Bonus 5 : Qu'est ce qu'une availability zone ?

Une zone de disponibilité ou AZ/Availability Zone, c'est une zone d'infrastructure détachées des autres en tout point critique (réseau, electrique, matériel, physique). Les AZ sont généralement libéllé par des datacenter (eu-west par exemple) et un numéro de salle (eu-west-3 par exemple).

