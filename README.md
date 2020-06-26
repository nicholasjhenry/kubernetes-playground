# Kubernetes Playground

- https://kubernetes.io/docs/home/
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://kubectl.docs.kubernetes.io
- https://docs.docker.com/docker-for-mac/kubernetes/

## Setup

1. Install [Docker for Mac (Edge)](https://docs.docker.com/docker-for-mac/edge-release-notes/)
   - https://docs.docker.com/docker-for-mac/#kubernetes
   - includes `kubectl`
2. Enable Kubernetes (context: `docker-desktop`)
3. Run the following Make target

  make dev.setup

## Tools

- kubectl
- kubectx

## Commands

Mostly based on https://kubernetes.io/docs/tutorials/kubernetes-basics/:

    # 1. Create a cluster (via Desktop Docker, Minikube, DigitalOcean)
    doctl account get
    kubectl version
    kubectl config get-contexts
    kubectl config use-context docker-desktop # or kubectx
    # cluster details
    kubectl cluster-info
    kubectl get nodes

    # 2. Deploy an app (normally handled by a configuration file)
    kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
    kubectl get deployments
    kubectl proxy
    curl http://localhost:8001/version

    # 3. Viewing Pods and Nodes
    # Step 1: Check application configuration
    kubectl get pods
    kubectl describe pods
    # Step 2: Show the app in the terminal
    export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
    curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME # cannot do /proxy on `docker-desktop`
    # Step 3: View the container logs
    kubectl logs $POD_NAME
    # Step 4: Executing command on the container
    kubectl exec $POD_NAME env
    kubectl exec -ti $POD_NAME bash
    cat server.js
    curl localhost:8080

    # 4. Exposing Your App
    # Step 1: Create a new service
    kubectl get services
    kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 # LoadBalancer not available
    kubectl get services
    kubectl describe services/kubernetes-bootcamp
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') # port exposed
    curl localhost:$NODE_PORT
    # Step 2: Using labels
    kubectl describe deployment
    kubectl get pods -l app=kubernetes-bootcamp
    kubectl get services -l app=kubernetes-bootcamp
    export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
    kubectl label pod $POD_NAME version=v1
    kubectl describe pods $POD_NAME
    kubectl get pods -l app=v1
    # Step 3: Deleting a service
    kubectl delete service -l app=kubernetes-bootcamp
    kubectl get services
    curl localhost:$NODE_PORT # fail
    kubectl exec -ti $POD_NAME curl localhost:8080 # still running internally

    # 5. Scale up your app
    # Step 1: Scaling a deployment
    kubectl get deployments
    kubectl get rs
    kubectl scale deployments/kubernetes-bootcamp --replicas=4
    kubectl get deployments
    kubectl get pods -o wide
    kubectl describe deployments/kubernetes-bootcamp
    # Step 2: Load Balancing
    kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
    kubectl describe services/kubernetes-bootcamp
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') && echo NODE_PORT=$NODE_PORT
    curl localhost:$NODE_PORT # repeat to see the different pod
    # Step 3: Scale Down
    kubectl scale deployments/kubernetes-bootcamp --replicas=2
    kubectl get deployments
    kubectl get pods -o wide

    # 6. Update the version of the app
    # Step 1: Update the version of the app
    kubectl scale deployments/kubernetes-bootcamp --replicas=4
    kubectl get deployments
    kubectl get pods
    kubectl describe pods # current image version
    kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
    # Step 2: Verify an update
    kubectl describe services/kubernetes-bootcamp
    export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') && echo NODE_PORT=$NODE_PORT
    curl localhost:$NODE_PORT
    kubectl rollout status deployments/kubernetes-bootcamp
    kubectl describe pods | grep Image
    # Step 3: Rollback an update
    kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
    kubectl get pods # v10 does not exist
    kubectl describe pods # for more info
    kubectl rollout undo deployments/kubernetes-bootcamp
    kubectl get pods
    kubectl describe pods

    # Cleanup
    kubectl delete deployment,service kubernetes-bootcamp

## Other helpful commands

    kubectl get events
    kubectl config view
    kubectl port-forward hello-3015430129-g95j6 8080

## Definitions

> Kubernetes automates the distribution and scheduling of application containers across a cluster in a more efficient way.

|         Term          |                                                                 Definition                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| application           |                                                                                                                                             |
| application instance  | a process for a running application container                                                                                               |
| cluster               | a group of computers to work as a single unit                                                                                               |
| container             |                                                                                                                                             |
| container runtime     | Examples: Docker, rkt                                                                                                                       |
| application container | an application deploy in a container to a cluster (containerized application)                                                               |
| deployment            | a manifest describing how to create and update containerized applications to run on individual nodes inside a pod, optionally via a service |
| deployment controller | a monitoring process in the master resource for a deployment                                                                                |
| label                 | Example: `app: foo`                                                                                                                         |
| `LabelSelector`       | a selector for a label                                                                                                                      |
| kubelet               | an agent for a node resource communicating with a master resource of the cluster.                                                           |
| master (resource)     | a co-ordinator resource                                                                                                                     |
| node (resource)       | a worker resource (VM or physical machine) with a container runtime                                                                         |
| pod                   | a host for application instances of one or more application containers and shared resources accessible via an IP address                    |
| pod lifecycle         |                                                                                                                                             |
| replica               | a duplicated set of pods to scale an application |
| `ReplicaSet`          |                                                                                                                                             |
| resource              |                                                                                                                                             |
| rolling update        | an update with zero downtime by incrementally updating pods |
| selector              |                                                                                                                                             |
| service               | a logical set of pods (deployment) and a policy by which to access them determined by a `LabelSelector`      |
| `ServiceSpec`         | a configuration for a service                                                                                                               |
| shared resource       | Examples: shared storage or volume, networking, shared configuration                                                                        |
| type                  | a field in a `ServiceSpec`. Examples: `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`                                               |
