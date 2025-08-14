# Continuous Deployment on Kubernetes (Demo)

## Prerequisites

The following utilities are required.
Links are provided the install documentation for each utility.

**Prerequisites**

- [Docker](https://docs.docker.com/engine/install/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [argocd CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

## Cluster Setup

Before deploying any applications, we will create a multi-node Kubernetes cluster using Minikube with Docker.
This will create a cluster using virtualized containers instead of physical nodes.

The following command will create a cluster `k8demo` with one control plane node and three worker nodes.

```bash
minikube start -p k8demo --nodes 4 --driver=docker
```

After provisioning the cluster, confirm that Minikube is running and all nodes are properly initialized.

```bash
minikube status -p k8demo
```

Once initialized, ensure kubectl is configured to interact with the newly created cluster.
This step is executed automatically by certain minikube commands, but is recommended anyway to avoid confusion.

```bash
kubectl config use-context k8demo
```

## Deploying Argo CD

Argo CD is installed in a dedicated namespace using the official kubernetes installation manifest.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

After installation, verify the created resources and wait for them all to be listed as ready.

```bash
kubectl get all -n argocd
```

Finally, enable access to the ArgoCD dashboard using port forwarding.
The following command will run in the foreground, exposing the Argo web interface on local port 8080.
This command will run in the foreground, so it is recomended to run it in a dedicated terminal session or
to pipe it into the background.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The web platform should now be accessible at port 8080.
To log in, use the username admin and retrieve the initial password with the command below.
Accessing the application may require waiving an `Insecure Connection` warning from your browser due to
the lack of a TLS certificate.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## Cluster Teardown

To tear down the Minikube cluster and remove all associated containers and data, use the following command:

```bash
minikube delete -p k8demo
```