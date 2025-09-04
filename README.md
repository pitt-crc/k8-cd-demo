# Continuous Deployment on Kubernetes (Demo)

This repository demonstrates the deployment of ArgoCD against a local Minikube cluster.

## Prerequisites

The following utilities are required.
Links are provided to the install documentation for each utility.

**Prerequisites**

- [Docker](https://docs.docker.com/engine/install/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [argocd CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

This demo will also include using git operations to trigger automatic deployments.
You will need a forked copy of this repository.

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

## Deploying an Application

Add the repository you forked as an approved source in ArgoCD:

1. Navigate to **settings > Repositories** and click **+ Connect Repo**.
2. Fill out the form using the values listed below, leaving all other fields blank.

| Field          | Value                              |
|----------------|------------------------------------|
| via            | `HTTP/HTTPS`                       | 
| type           | `git`                              | 
| Repository URL | The URL of your forked repository. | 

With the repository added, configure the project permissions to allow Argo CD to deploy manifests from this repo to the
namespace you created:

1. Navigate to **Settings → Projects** and click **New Project**.
2. Enter a descriptive project name and click **Create**. You will be redirected to the project settings page.
3. Under **Source Repositories**, add your forked repository.
4. Under Destinations, add the cluster server and the namespace `demo-*` (note the wildcard).
5. Under Cluster Resource Allow List, add an entry for kind `Namespace` with an empty group.

Finally, deploy the application:

1. Navigate to **Applications** and click **+ New App**.
2. Select the project you created. The
   form will be limited.

Finally, deploy the application:

1. Navigate to Applications in the Argo CD dashboard.
2. Click + New App and fill out the form using the values provided below.
   The dropdown options will be limited to those included in the selected project.

| Field                 | Value                                    |
|-----------------------|------------------------------------------|
| Application Name      | `nginx`                                  | 
| Project Name          | The name of the project you just created | 
| Sync Policy           | `Automatic`                              | 
| Prune Resources       | Checked                                  | 
| Self Heal             | Checked                                  | 
| Auto-Create Namespace | Checked                                  | 
| Repository Url        | The repository you configured in argocd  | 
| Revision              | `HEAD`                                   | 
| Path                  | `manifests`                              | 
| Cluster Url           | Select `Name` anc select `in-cluster`    | 
| Namespace             | `demo-nginx`                             |

Argo will automatically create the namespace and provision the resources defined in the manifest.
If you're wondering what happens to the namespace when an app is destroyed, the answer is nothing.
Argo will not delete the empty namespace (see [this relevant issue](https://github.com/argoproj/argo-cd/issues/7875)).

## Synchronize Changes

For applications with an automatic synchronization strategy, ArgoCD will automatically poll git and pull and manifest
changes.
By default, this polling occurs every 3 minutes.

To trigger a change, edit the example manifest under by increasing the number of replicas in the following line:

```yaml
  replicas: 2
```

Commit/push your changes to the upstream repo and wait for Argo to pick up the changes.
You can also manually trigger a sync by navigating to the app and clicking `Sync`.

## Cluster Teardown

To tear down the Minikube cluster and remove all associated containers and data, use the following command:

```bash
minikube delete -p k8demo
```

## Future Work

Adding RBAC:

- [argo docs](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/)
- [Medium article on LDAP](https://medium.com/@dast04/setting-up-argocd-ldap-authentication-rbac-2024-50c0d8135713)
