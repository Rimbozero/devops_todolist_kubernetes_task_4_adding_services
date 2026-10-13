# Testing the ToDo application on Kubernetes

The commands below assume `kubectl` is configured for your cluster and are run
from the repository root.

## Deploy the application and services

Create the namespace, then apply the two application pods and both services:

```sh
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/todoapp-clusterip-service.yml
kubectl apply -f .infrastructure/todoapp-nodeport-service.yml
```

Wait for both pods to become ready:

```sh
kubectl get pods -n todoapp -w
```

The `todoapp` ClusterIP service selects both pods using the `app: todolist`
label and distributes connections to ready endpoints.

## Test the ClusterIP service from BusyBox

Start the BusyBox pod and wait until it is running:

```sh
kubectl apply -f .infrastructure/busybox.yml
kubectl wait --for=condition=Ready pod/busybox -n todoapp --timeout=120s
```

Call the service by its DNS name from inside the cluster:

```sh
kubectl exec -n todoapp busybox -- curl -i http://todoapp:8080/api/health
```

The fully qualified DNS name is
`todoapp.todoapp.svc.cluster.local`; from the `todoapp` namespace, `todoapp`
is the short service DNS name. A successful health check returns `Health OK`.

## Test the ToDo application with port-forward

Forward the ClusterIP service port to your local machine:

```sh
kubectl port-forward -n todoapp service/todoapp 8080:8080
```

While that command is running, open
[http://localhost:8080](http://localhost:8080) in a browser. The API can also
be checked at [http://localhost:8080/api/health](http://localhost:8080/api/health).
Stop the port-forward with `Ctrl+C`.

## Access the application through the NodePort service

The NodePort service is configured to use port `30080` on each Kubernetes node.
Find a node address:

```sh
kubectl get nodes -o wide
```

Open `http://<NODE-IP>:30080` in a browser, replacing `<NODE-IP>` with an
address reachable from your machine. For a local cluster, use the node address
provided by that cluster (for example, `minikube ip` for Minikube).
