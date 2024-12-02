# Devops K8S - Day 1

# Step 1
## Quelles sont les informations que l'on retrouve dans ce fichier ?
Le fichier de configuration contient plusieurs informations qui vont nous intéresser:
- la version de l'api utilisé pour créer les objects k8s
- les clusters qui contient un seul cluster qui contient lui-même plusieurs infos
    - le certificat
    - le serveur qui va être notre controle plane
    - le nom de ce contrôle plane
- il y a également des contexte, dans notre cas nous n'en avons qu'un seul et il contient:
    - le nom du cluster ciblé 
    - le namespace associé, ce qui permet d'organiser nos clusters
    - le user qui permet de réaliser les actions que l'on va demander
    - le nom 
- on a également un current-context qui permet d'indiquer au kubectl quel contexte et quel cluster utiliser.

## Quelle est la différence ?
Dans le cas de la première commande, le namespace n'existe pas. Donc kubectl ne peut pas nous donner les informations de celui-ci.
Le namespace default (prenom-nom) lui existe et donc on peut récupérer les informatiosn de celui-ci.

> Note: Pour la suite, je vais utiliser k9s. Je suis plus à l'aise avec mais cela fonctionne avec l'API kubectl

## Quelles sont les propriétés principales que l'on retrouve ? (après avoir run un pod)
- apiVersion: l'API k8s à utiliser
- kind: le type de ressources
- metadata: on objet qui contient beaucoup d'informations sur notre pod (on ne va regarder que les plus intéressantes):
    - le name et le namespace, pour identifier et organiser nos pods (on a également un uid pour l'identifier)
    - label: permet de regrouper des pods entre eux via des selector. 
    - on remarque aussi une annotation (kubernetes.io/limit-ranger) pour limiter les ressources mémoire, proceseur, etc.
- les spec:
    - les containers
        - avec une image, une imagePullPolicy qui détermine si on doit pull l'image ou non
        - un nom, des volumes (nommés et avec des propriétés)

## Que se passe-t-il lors de cette deuxième création en impératif ?
On obtient une erreur car le pod existe déjà. On ordonne la création d'un pod sans regarder s'il existe déjà
> kubectl run mynginx --image registry.takima.io/school/proxy/nginx

## Que se passe-t-il lors de cette deuxième création en déclaratif ?
Dans le cas de l'imprératif, k8s va conformer l'état de notre pod à ce qu'il devrait être (observed state = desired state), il va enregistrer le fichier yaml en tant que ressources et vérifier que le pod reste dans l'état souahité.
> kubectl apply -f ...

## Que remarquez-vous dans la description des properties spec: template ? À quoi sert le selector: matchLabels ?
Le spec template permet de définir comment les pods du replicat set doivent être. 
Le selector permet d'indiquer à k8s quels pods doivent constituer le replicat set, quels pods il va devoir créer et lesquels il doit surveiller pour qu'ils soient dans l'état désiré. 
Tous les pods avec le label associé seront géré par notre replicat set.

## Combien y a-t-il de pods déployés dans votre namespace ?
Il y en a 3 désormais (les 3 de mon replicat set).
> k9s
> kubectl get pods

## Que se passe-t-il ? (suppression d'un pod)
K8s va recréer le pod que j'essaye de supprimer, car il constate que le observed state est différent du desired state. Il va donc recréer un pod automatiquement.
> kubectl delete pods ... 

## Que se passe-t-il ? (supprimer un replicat set)
Avec la commande `kubectl delete -f unicorn-front-replicaset.yml`, les 3 pods sont supprimés.

## Quels sont les changements par rapport au ReplicaSet ?

Le deployement permet un plus haut niveau d'abstraction et permet un mechanisme de rolling update, rollbacks sans down time.
Il permet aussi de versionner nos applications.

## Combien y a-t'il de ReplicaSet ? De Pods ?
- 1 deployment
- 1 replicat set
- 3 pods
> kubectl get all

## Une fois terminé, combien y a-t-il de replicaset ? Combien y a-t-il de Pods ? Allez voir les logs des événements du déploiement avec kubectl describe deployments. Qu’observez-vous ?
- 2 replicat set
- 3 pods
- 1 deployment 
> kubectl get all

On observe les scale up et scale down pour maintenir les 3 pods. (j'ai mis une strategy de rolling update avec un maxUnavailable à 1)

## Combien y a-t-il de révisions ? À quoi correspond le champ CHANGE-CAUSE ?

CHANGE-CAUSE correspond à la commande qui a initié le changement dans notre deployement que nous annulons.
Dans mon cas je vois 3 révisions car il y a des erreurs mais normalement je dois avoir:
- 1 changement de version
- 1 chagement d'image pour mettre l'image en erreur

> kubectl rollout history deployment.v1.apps/unicorn-front-deployment --revision=2

## Combien y a-t'il de Pods?
Avec la mise à jour du nombre de replicas, on a désormais 5 pods
> kubectl get pods
> Après avoir fait: kubectl scale deployment.v1.apps/unicorn-front-deployment --replicas=5

## Que se passe-t-il au niveau ReplicaSet ? (mise en standby)
- 3 replicaset
> kubectl get all

## Que se passe-t-il au niveau ReplicaSet ? (après resume)
- 4 replicaset
> kubectl get all

# Bonus
## Que constatez-vous ? Pourquoi ? (memory-limit)
Le pod est kill car il est en OutOfMemory
> k9s

## Que constatez-vous ? Pourquoi ?
Le pod ne marche pas car il demande plus de ressources qu'il n'en a selon son fichier yaml.
> k9s

# Step 2
# Bonus
> Clean name space:
> kubectl delete --all deployments && kubectl delete --all services && kubectl delete --all ingress

## Que se passe-t-il ? Pourquoi ?
L'image ne peut pas être pull car nous n'avons pas accès au registry. Les pods sont donc en erreur.

## Décrivez ce que répond la Web App ? Actualisez votre page avec CTRL + F5. Que se passe-t-il ?
"Je suis le taki-pod situé sur le noeud avec l'ip"
Le fond change de couleur à chaque requête.

## Que constatez-vous sur le navigateur ?
L'IP, le nom du pod et nom du node sont désormais renseignés.

# Step 3
## Bonus 1 : Rendre sa couleur secrète
```yml
        - name: CUSTOM_COLOR # Vrai key de la variable d'env. Peut-être différent de la valeur dans le configmap
          valueFrom:
            secretKeyRef:
              name: hello-secret  # Nom du configmap
              key: color     # nom de la clef dans le configmap

```

## Bonus 2 : Mon pod est-il en vie ?
```yml
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
```

## Bonus 3 : Let my kubernetes Scale !!!
### Observez ce qu'il se passe. Que constatez-vous ?
Avec 6 replicas, on arrive à 50% de CPU donc on ne créé pas plus de pods.
On voit que l'on garde les 6 replicas après que le charge CPU soit retombée à 0%, il faut attendre 5 minutes pour que ceux-ci retombent à 1.


## Bonus 4: Control my traffic !!!
### Que constatez-vous ?
On ne peut plus joindre notre pod avec la nouvelle policy. 
Car notre label ne match pas avec la policy

## Bonus 5: Run on one AZ
### Qu'est-ce qu'une Avaibility Zone au niveau infrastructure ?
Une Availability Zone est une unité d'isolation au sein d'une Région Cloud, conçue pour garantir la haute disponibilité et la tolérance aux pannes. 
Une région peut contenir plusieurs AZ.
Isolation physique** :
    - Chaque AZ dispose de ses propres centres de données, alimentations électriques, et systèmes de refroidissement.
    - Séparation physique pour réduire les risques liés aux catastrophes.
Connectivité:
    - Réseau privé à haut débit et faible latence entre les AZ d'une même région.
Redondance:
    - Infrastructures et systèmes réseaux redondants dans chaque AZ.
Disponibilité accrue:
    - Répartition des ressources sur plusieurs AZ garantit la continuité des services.

### Faire en sorte de tourner sur une seule AZ, par exemple eu-west-3a
Checked
## Bonus 6: Run one pod on each server
On utilise un daemonset pour que sur chacun des workers.
On a 6 workers (nodes)

## Bonus 7: Change my rolling update policy
### que se passe-t-il au niveau de vos pods pendant l'update ?

Tous les pods sont down puis recréés dans un nouveau replicaset. Cela donne l'avantage d'avoir la nouvelle version disponible en une fois mais cela provoque aussi d'avoir du downtime
