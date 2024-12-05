# Day 2
# Step 1
## Que se passe-t-il au niveau des Pods de l’API ? Vous pouvez jeter un œil aux logs. (kubectl logs -f nomdupod)

L'API ne peut pas démarrer sans la base de donnée. 
La base de donnée nécessite les variables d'environnement.

# Step 2
> k exec pods/database-deployment-77767bc6b7-fkktr -ti -- psql -d cdb-db -U postgres

On peut lister les tables avec `\l` 

# Step 3
## Quel est le nom du service de la base de données ?

Dans mon cas, le nom du service pour la base de données est `pg-service`, car mon container pour la db est pg et le fichier du service se nomme `pg-service.yml`

# Step 4

# Step 5
## Pourquoi plus rien ne fonctionne ? Pourquoi faut-il kubectl rollout restart deployment MON_API

Plus rien ne marche car la bdd n'est pas persisité. Cela rend donc l'utilisation de l'API impossible.

# Step 6 
## D’après le tableau, quel est le type d’accès implémenté par notre Storage Class EBS ? Pourquoi cela convient parfaitement pour la persistance de la base de données Postgres ?

Dans notre cas, on va avoir besoin de lire de la donnée et d'en écrire pour ajouter des enregistrements. Donc `ReadWriteOnce` car un seul pod va lire et écrire. 
Cela correspond bien à postgres car il va être le seul à intéragir avec cette db. 
On va donc s'orienter vers AWSElasticBlockStore sachant que nos serveurs sont déjà des serveurs AWS.

# Bonus 1 : Administration de la base de données
> k port-forward svc/pgadmin 8080:80

# Bonus 2 : Les StatefulSets

le statefulset permet de générer automatiquement le PVC. Le nom du PVC généré est `pg-data-pg-statefulset-0`.

# Bonus 3 : Operator Postgres

Operator: permet de faire le lien entre les ressources custom et l'API k8s. Il agit comme un controller pour comparer le desired state et le observed state.

Un des deux pods créé: un leader et un standby. Le standby permet de redonder le leader en cas de problème.
Si on kill le pods qui est le master, alors le pod en standby devient le master. 

avec k9s, on peut voir que le cron a bien tourné.
