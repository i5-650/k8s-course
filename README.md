# K8s-courses

## The basics

There are multiple `kind`s in k8s with various purpose:

- Create multiple pods (identical) -> replicaset
- Handle pods (number, lifetime, etc) -> deployment
- Loadbalance and expose internaly a pool of pod -> service

---

## Services

- ClusterIP (default): for services that won't be exposed to the outside world.
- NodePort: an extension of the ClusterIP that allows external connectiviy.  
- LoadBalancer: connect our applications externally, is used for HA and scaling.
- ExternalName: it's a special case that doesn't use a selector but a DNS name.

Service example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-postgres-service
spec:
  selector:
    app: app-postgres 
    # if type is not present: it will be a ClusterIP by default  
  type: ClusterIP # NodePort | ExternalName | LoadBalancer
  ports:
    - protocol: TCP # - name: HTTP for HTTP streams
      port: 5432
      targetPort: 5432
```

**Q:** How does a service loadbalance on a pool of pods ?
What do we need in our deployment / replicaset /etc ?

**A:** You need to match the label on the deployment and selector. Match means "contains".
For instance, my deployment would need to have my `label` set to `app: app-postgres-v14`.

---
**Q:** Where do we define what ports are exposed and what do we define ?

**A:** Just like in a `docker-compose.yml`, you can define a `targetPort` and a `port`.
Other pods will use the `port` to connect with our pods.

---
**Q:** How do we expose our Pods to the internet ?

**A:** We use an `ingress` ! In this class, we used the `IngressClassName: nginx`.
You can use different `Ingressclass` without any conflicts.

---

## Ingress

Ingress are used to expose services to internet.

**Q:** How do we indicate to which service we route the request ?

**A:** We the `backend` which is the service with a name and a port.
Example: app-postgres-service for the selector and `5432` for the port.

---
**Q:** What if an API want to access a database ?

**A:** Unless you want to publicly expose your database, you won't need an Ingress.
So, we define the `DB_URL` which will be composed of the service's name and the port.
Example: `app-postgres-service.mydomain.com:5432`.

> Note: This works because there is a sort of internal DNS in k8s.
> In our class, we used the Container Network Interface: [Calico](https://docs.tigera.io/calico/latest/about/).

---
If we want to exchange between namespaces: we indicate the namespace.
Example: `service-name.namespace`.

### Example And Break Down

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: api-ingress
spec:
 ingressClassName: nginx
 rules:
 - host: api.mydomain 
   http:
     paths:
     - backend:
         service:
           name: api-service
           port:
             number: 8080
       path: /
       pathType: Prefix
 tls:
 - hosts:
   - api.mydomain
   secretName: app-wildcard 
```

- `backend.service`: the service we want to redirect to.
- `backend.service.port`: the target port of the service.
- `host`: the DNS name of our app.
- `tls` (optional): with the secret or with `letsencrypt`.
- `path`: to route request based on the HTTP path.

---

## Deployments and others

There are deployments but there are also:

- Statefulset: to have a Persistent Volume Claim (PVC).
- DaemonSet: to have a pod on every node. (e.g. For observability with [Prometheus](https://prometheus.io))

---

## Operator

An operator is a Custom Resources Definition (CRD).
It means that you crate your own `kind`.
It also means that you have your own controller for that resource

---

## Persistence

To have a database, we need to persist it. You will also have to use a statefulset.
In order to have persistence, you need one of those:

- A Persistent Volume (PV).
- A Persistent Volume Claims (PVC): The PVC will create the PV for us via a class.

Volume have access modes:

- ReadOnly: ...
- ReadWriteOnce: Only on pod can read and write.
- ReadWriteMany: Many pods can read and write.

---

## Templating with Helm

To create template of your k8s environment. It's a sort of package manager for k8s.

Helm has two modes:

- templating: you want to deploy your apps, only for you.
So you have a few values and the rest is hard coded.
- packaging: you want to distribute your software.
You put into variable as much as you can.

**Q:** How to run the helm chart ?

**A:** You can use `helm install <name> path/to/chart`.
> Note: We tend to use `helm upgrade --install ...` instead.
---

**Q:** How do I test my helm configuration ?

**A:** You can use the `helm template` command.

---

## Argo CD

Argo CD is a continuous delivery tool for k8s.
Argo is part of the GitOps field and work in pull mode.

- push mode: run the code and apply it to our K8S. (e.g: ansible, terraform)
- pull mode: retrieve the code and apply it to our K8S (e.g: ArgoCD)

**Q:** What do we create in Argo CD ?

**A:** We create an application from a repository.
This repository must have a helm chart and all the values.

---
**Q:** What does Argo do ?

**A:** Argo sync the git with the infra. The infrastructure must match the git.
There is two way to sync those:

- autosync: each git change trigger a sync. 1 commit = 1 deployment.
- not autosync: the git pipeline will call the Argo API and trigger the sync.

---

## Advanced Scheduling

There is some concept that will help you:
Affinity and Non-affinity:

- affinity: groups pods or place them in near servers. (e.g: high couplage front-back)
- anti-affinity: avoid to have pods on the same server. (e.g: database and a back-end)

Availability Zone (AZ):
it's a zone of the cluster that is isolated from the others.
It aims to define a zone that is autonomous.
Examples:

- two servers in a same rack
- two different rack
- two server rooms
- two datacenter
- ...

> Note: The servers still need to be closed because you want a low latency.
As low as possible

If you want to have more extended zone / far away, you might need different regions.
> Note: In AWS, volumes are associated to a datacenter.
Therefore, to an AZ.

---

## General questions

**Q:** how to create a resource in K8S ?

**A:** We use the API on the controller node! There is two options:

- declarative: you create YAML that you send to the API.
- imperative: you use commands to create the resources.

> Note: The controller node also has :

- the [ETCD](https://etcd.io) database.
- the scheduler.
- the controller manager (which has the cloud controller).

> Note: The declarative way is the most common one.

---
**Q:** What does the worker have ?

**A:** The workers only have two things: the container engine and the `kubelet`.

---
**Q:** What are the important parts of a deployment ?

**A:** The deployment contains multiple informations:

- `spec.replicas`: the number of replicas.
- `spec.template.spec.containers.image`: the image used.
- `metadata.name`: the deployment name.
- `metadata.label`: the labels (they are use by selector)
- `spec.template.spec.containers.resources`:
  - `request`: CPU and memory requirement to schedule a pod on a node.
  - `limit`: CPU and memory maximum. If exceeded, the pod is killed. (CPU Throttle)
  - if you set a limit above the server limits, you are doing "over-provisioning"
- `spec.template.spec.containers.*Probe`:
  - `readinessProbe`: check that the pod is ready to be in the service pool.
  - `livenessProbe`: check that the pod is able to answer. If not, we kill it.
  - `startupProb`: check that the pod started. If it takes too long, we kill it.
- `spec.secuirtyContext`: security rules to prevent hack. (e.g: non-root user).
- `spec.template.spec.containers.volumeMounts`: the persistence mapping.  

---
**Q:** How do we handle application configuration ?

**A:** We have two options: `ConfigMap` and `Secret`.
They both are environment variable that are injected in the pods.
However, `Secret` are hidden / encrypted on most K8S distribution except on Vanilla.
> Note: If the pods don't refresh its environment variable:
you need to do it yourself with `annotations`

---
**Q:** When you have a deployment, how do you handle the High Availability ?

**A:**  You use Horizontal Pod Autoscaler (HPA).
There is a special command `kubectl autoscale ...` to create an autoscaler.
It increase the number of pods to scale horizontaly.

---
**Q:** By default, is the communication between pods open or closed ?

**A:** By default, everything is opened.

- ingress: input = who can access my pods.
- egress: output = who the pod can access.

---
**Q:** What is the CNCF ?

**A:** Cloud Native Computing Foundation.

---
**Q:** How to monitor your cluster ?

**A:** You can use [Prometheus](https://prometheus.io) to harvest data.
Then use [Grafana](https://grafana.com) to have dashboards.

---
**Q:** What are the update strategy in a deployment ?

**A:** There are a few and we only saw two of them:

- RollingUpdate: Update without any service interruption.
It replace one pod at a time to guarantee the availability.
- Recreate: kill all old pods and recreate them all.
It allows you to not have any old version of you application.

---

## Comments

In our API, what was in charge of creating the data was [Flyway](https://www.red-gate.com/products/flyway/community/).
It ran at the API start-up.
