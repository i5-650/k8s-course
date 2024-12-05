# K8s-Courses
## Gestion des Pods

ReplicaSet : Permet de gérer plusieurs Pods identiques.
Deployment : Gère plusieurs Pods et leur cycle de vie.
Service : Permet de load balancer et d'exposer un pool de Pods à l'interne.

## Types de Services

- NodePort : Expose le service sur un port fixe de chaque nœud du cluster.
- ClusterIP : Expose le service à l'interne du cluster.
- LoadBalancer : Expose le service avec un équilibreur de charge externe.
- ExternalName : Associe un nom DNS externe à un service.

## Configuration d’un Service

Pour qu’un service load balance sur un Pod :
1. Match selector : Les labels des Pods doivent correspondre au sélecteur défini dans le service.
Exemple :
```yml
    selector:
      app: postgres
    labels:
      app: postgres-v16
```
2. Ports dans un Service

    Target Port : Port exposé par le Pod.
    Port : Port exposé par le service, utilisé par d'autres services ou Pods pour accéder au Pod.

## Exposition sur Internet : Ingress

Ingress : Permet d’exposer un service sur Internet.
IngressClass : Exemple utilisé : nginx.
Avantage : Plusieurs IngressClass peuvent coexister sans conflit.

### Fonctionnement d’un Ingress

- Spécification du backend : Nom du service et son port.
- Options d’un Ingress :
    - Service : Nom du service cible.
    - Port : Port du service.
    - Host : Nom DNS de l'application.
    - TLS : Activer TLS avec un certificat (secret ou Let’s Encrypt).
    - Path : Chemin d'accès. (dans notre cas on a mis `/` tout le temps)

### Communication entre API et base de données

Dans le champ db_endpoint, utilisez le nom du service + port.
Cela fonctionne grâce au DNS interne de Kubernetes.
Dans le cas où on souhaite contacter un domaine en dehors de notre namespace il faut préciser le namespace.
Exemple : service-name.namespace.

### Autres types de déploiements

StatefulSet : Gère des Pods avec un stockage persistant (PersistentVolumeClaim - PVC).
DaemonSet : Déploie un Pod sur chaque nœud du cluster (exemple : observabilité avec Prometheus).

## Stockage persistant

- PersistentVolume (PV) : Ressource de stockage.
- PersistentVolumeClaim (PVC) : Requête pour obtenir un PV.
    - Un PVC peut automatiquement créer un PV via une classe de stockage.
    - Possibilité de définir manuellement un PV.

### Modes d'accès aux volumes

- ReadOnly : Lecture seule.
- ReadWriteOnce : Un seul Pod peut lire et écrire (sauf s’ils sont sur le même serveur).
- ReadWriteMany : Plusieurs Pods peuvent lire et écrire.

## Templating et gestion des configurations

Helm :
    - Templating : Usage pour soi-même, avec un fichier values minimal.
    - Packaging : Distribution d’une application packagée pour le partage.
Création de ressources Kubernetes :
    Méthodes :
        - Déclaratif (via des fichiers YAML).
        - Impératif (via des commandes CLI).
    Interaction avec l’API Kubernetes (via les nœuds contrôleurs qui contiennent l’ETCD, le scheduler et le controller manager).

## Configuration d’une application

ConfigMap : Injection de variables d'environnement non sensibles.
Secret : Injection de données sensibles sous forme chiffrée. (dans le cas de certains distributions de k8s comme vanilla, il est possible que les secrets ne soient pas chiffrés)

## Ressources avancées

Operator :
    Définit une ressource personnalisée (CRD).
    Installe un contrôleur pour gérer cette ressource.
Custom Resource Definition (CRD) : Permet de définir des types de ressources spécifiques.

## Déploiement et Scaling
### Définition d’un Deployment

Éléments clés :

- Nombre de réplicas.
- Image Docker.
- Métadonnées (nom, labels, annotations).
- Ressources :
    - Requests (CPU/mémoire) : Minimum requis pour le Pod.
    - Limits (CPU/mémoire) : Maximum autorisé.
- Probes :
    - Readiness : Vérifie si le Pod est prêt à recevoir du trafic.
    - Liveness : Vérifie si le Pod est vivant.
    - Startup : Vérifie le démarrage (si trop long, le Pod est tué).
- SecurityContext :
    - Pas d’exécution en tant que root.
    - Définit un utilisateur spécifique.
- Mapping de volumes persistants.

### Scaling

HPA (Horizontal Pod Autoscaler) : Permet d’ajuster automatiquement le nombre de Pods en fonction de la charge.

## Sécurité réseau

Par défaut, les communications entre Pods sont ouvertes.
Network Policies :
    - Ingress : Règle les accès entrants.
    - Egress : Règle les accès sortants.

## Observabilité

Prometheus : Moteur de collecte de métriques (timeseries).
Grafana : Visualisation des métriques.

## GitOps

ArgoCD :
    Crée une application (liée à un dépôt Git contenant les chart Helm).
    Synchro GitOps : Aligne l’infrastructure avec l’état déclaré dans le dépôt.
    Modes :
        - Autosync : Tout commit entraîne une synchro automatique (moins conseillé pour les releases).
        - Manuel : Déclenche la synchro via l’API depuis la pipeline CI/CD.

## Approches GitOps

Push : Ex. Ansible/Terraform. Applique directement les modifications sur le cluster.
Pull : Ex. ArgoCD. Récupère les modifications depuis le dépôt et les applique.

## Autres concepts avancés

Affinités :
    Affinity : Regroupe des Pods ou les place sur des serveurs proches.
    Anti-Affinity : Évite que deux Pods spécifiques soient sur le même serveur.
AZ (Availability Zone) :
    Cluster isolé géographiquement mais avec une faible latence interne.
Régions : Zones éloignées avec une latence plus élevée (ex. plusieurs datacenters).

## Policies de déploiement

Rolling Update : Mise à jour sans interruption (remplacement progressif des Pods).
Recreate : Supprime tous les Pods avant de recréer les nouveaux.

Note : Lorsque notre API démarre, Flyway applique automatiquement les migrations de la base de données.

