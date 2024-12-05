# Day 3

# ECK

## Que constatez-vous ?
On constate que nous avons 3 pods. Car nous avons scale up nos elastics.
```
epoch      timestamp cluster                status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1733301373 08:36:13  elasticsearch-operator green           3         3     22  11    0    0        0             0                  -                100.0%
```

```
shards disk.indices disk.used disk.avail disk.total disk.percent host         ip           node
     8       38.2mb    55.4mb    917.9mb    973.4mb            5 10.60.45.78  10.60.45.78  elasticsearch-operator-es-default-0
     7       35.3mb    52.5mb    920.8mb    973.4mb            5 10.60.16.70  10.60.16.70  elasticsearch-operator-es-default-1
     7        3.6mb      21mb    952.4mb    973.4mb            2 10.60.14.230 10.60.14.230 elasticsearch-operator-es-default-2

```

# Helm

# ArgoCD

Mise en place d'une application de production sur ArgoCD
![alt text](./argocd-prod.png)


### Pour vérifier que tout fonctionne, essayez de détruire un deployment manuellement dans votre Cluster. Que se passe-t-il ?

Lorsqu'on essaye de détruire un deployment manuellement, ArgoCD redeploy automatiquement la ressource kubernetes.

### Essayez de modifier le values.yaml en augmentant le replicaCount par exemple. Que se passe-t-il ?

Quand on modifie le replicaCount et qu'on push sur gitlab, les applications vont se resynchronisé automatiquement et ArgoCD va déployer le nombre de replicat actualisé par rapport au replicaCount.
