INSTALLATION
---------------

Install these tools, to start using clusters:

    a. minikube
    b. docker desktop with enabled kuber

USAGE
-----

1. Run the cluster locally: `minikube start`
2. To stop the cluster: `minikube stop`
3. To add ingress integration: `minikube addons enable ingress`
4. Deletes a local Kubernetes cluster. This command deletes the VM, and removes all associated files.`minikube delete`
5. To run a web dashboard: `minikube dashboard`
6. Returns a local ingress' ip: `minikube ip`
7. Addons list: `minikube addons list`
8. Check if the pulling from a repository working `minikube ssh docker pull docker.io/esase/news-api:1.0`
9. Login into the docker `minikube ssh docker login`

RUNNING
-------

1. to be able to forward traffic from local machine to the cluster - `minikube tunnel` (and your ingress resources would be available at "127.0.0.1")

ADVANCED KUBER COMMANDS:
-----------------

1. Apply a new config to the kuber cluster: `kubectl apply -f ingress.config.yaml`
2. Get ingress info `kubectl get all -n ingress-nginx`
3. Get pods IP addresses: `kubectl get pod -o wide`
4. Get linked services with pods: `kubectl get endpoints`
5. Get a list of running services: `kubectl get svc -A`
6. Get list of system running pods: ` kubectl get pod -n kube-system`
7. Get a list of running services, pods and deployments: `kubectl get all -A`
8. Scale a deployment adding extra PODS `kubectl scale deploy myc2-wallet-api  --replicas=2`
9. Describe the service and it pods `kubectl describe service news-api`
10. Get advanced POD information  `kubectl get pods -o wide`
11. Delete resources (deployments, pods, services) `kubectl delete -f FILE_NAME`
12. Get all pods in all namespaces `kubectl get pods -A `
13. Get all pods in a concrete namespace `kubectl get pods -n bar-api`
14. Get list of all nodes `kubectl get nodes` 
15. Get a list of a replica set `kubectl get replicaset -A`
16. Delete all components and ingress rules by a namespace `kubectl delete all,ingress,secrets --all -n=news-api`
17. Get all ingress http rules `kubectl get ing -A -o json | jq -r '.items[].spec.rules[].http.paths[]'`
18. Enter a container `kubectl exec -it -n=foo-api foo-api-7555dcdb9c-lpfts foo-app -- /bin/bash`
19. Get a running container in a pod `kubectl get pods news-api-54bc64b954-28zdq -o jsonpath='{.spec.containers[*].name}' -n=news-api`
20. Show a secret with decoding`kubectl get secret docker-registry -o jsonpath="{.data.dockerconfigjson}" | base64 --decode`
21. Show all secrets `kubectl get secrets`
22. Describe the secret `kubectl describe secret mysecret`
23. Get a secret `kubectl get secret docker-registry -o json`
24. Get logs `kubectl logs news-api-7496877d7c-62qmq  -n=news-api -c news-api --tail=20 --follow`
25. Describe a pod `kubectl describe pods news-api-79b9594c46-8dbjf -n=news-api`
26. Get current context `kubectl config current-context`
27. Checkout context `kubectl config use-context myc-new-env`
28. Get nodes `kubectl get nodes -o wide`

RUNNING ON AWS:
--------------

1. Set a default profile `export AWS_PROFILE=esase`
2. Create a cluster using the `eksctl` tool
```
eksctl create cluster \
--name test-cluster \
--region eu-central-1 \
--nodegroup-name linux-nodes \
--node-type t2.micro
--nodes 3
```

4. Change the number of nodes in the cluster  `eksctl scale nodegroup --region eu-central-1 --cluster=test-cluster --nodes=5 --name=linux-nodes --nodes-min=2 --nodes-max=10`

5. Deleting cluster `eksctl delete cluster --region eu-central-1 --name prod-cluster`

DOCKER
-----

1. RUn `docker run -p 8080:8080 --rm -it esase/news-api:1.4`

2. build and publish `docker build -f ./infra/docker/Dockerfile -t esase/news-api:1.7 .`
  `docker push esase/news-api:1.7`

HELM
----

1. Installing ingress  https://kubernetes.github.io/ingress-nginx/deploy/


LINKS
-----

1. https://medium.com/@javatechie/kubernetes-tutorial-install-run-minikube-in-mac-os-k8s-cluster-369b25b0c3f0
2. https://minikube.sigs.k8s.io/docs/start/
3. https://stackoverflow.com/questions/58561682/minikube-with-ingress-example-not-working
4. https://github.com/vplauzon/aks/blob/master/ingress-multiple-ns/ingress2.yaml
5. https://gitlab.com/nanuchi/youtube-tutorial-series/-/blob/master/configmap-and-secret-volumes/mongodb-config-components.yaml
6. https://www.datree.io/resources/kubernetes-troubleshooting-fixing-imagepullbackoff-state-error
7. https://eksctl.io/usage/creating-and-managing-clusters/
8. https://github.com/weaveworks/eksctl/tree/main/examples
9. https://aws.amazon.com/premiumsupport/knowledge-center/eks-worker-node-actions/
10. https://www.youtube.com/watch?v=9EVs5LcaUcs
11. https://stackoverflow.com/questions/73398714/docker-fails-when-building-on-m1-macs-exec-usr-local-bin-docker-entrypoint-sh

SPECIFICATION:
-------------

0. `Minicube` is a tool for running a kubernetes cluster one a one machine for development purposes which use a virtual box for running pods.

1. All communication between `PODS` which are like abstract containers, with dynamic IP addresses, which could be deleted and restored many many times
on Nodes (which are physical or virtual machines) performed by using services. Which act like a glue between PODS, services know where PODS are located and how
to reach them (using a round robin algorithm, which simply selects a random POD if you are running several PODS)

2. There are several types of services: 
    a. `ClusterIp` which is default and the most popular one, where you simple assign it to a pod, using a selector of a POD, like:
    ```
    kind: Service
    apiVersion: v1
    metadata:
    name: bar-service
    spec:
        type: ClusterIP
        selector:
            app: bar <------
    ```
    All the `ClusterIp` services have a static IP address in the cluster, which are not changed, during the life time and has a load balancing stuff
    which selects randomly a POD, and the service cannot be exposed outside, and only is allowed inside the cluster.

    b. `Headless` service, which allows you to communicate with a specific `POD` which are not randomly selected. The use case, might be:
    mysql master slave communication. Where the worker needs to be synchronized with the master `POD`.
    For such services we should not specify the cluster ip (which is static in the cluster):
    ```kind: Service
        apiVersion: v1
        metadata:
            name: bar-service
        spec:
            clusterIp: None
            selector:
                app: bar
    ```
    which means the POD ip will be used instead, which can be used for a direct communication

    c. `NodePort` service allows to be called outside (it's like an alternative to the Ingress which is shipped out the box)

    ```
    kind: Service
    apiVersion: v1
    metadata:
    name: bar-service
    spec:
        type: NodePort <-----
    ports:
        # Default port used by the image
        - port: 3000 <---internal port in the cluster
        - targetPort: 3001 <-- where to redirect incoming requests
        - nodePort: 3008 <--- external port number     
    ```

    PS: there is a predefined ranges of ports which can be used: 30 000 to 32 767, the main disadvantage here in comparison with the ingress,
    that we need to define a lot `NodePort` bases services, to be able to use them outside. And this service usually is not used in the prod
    by the security reasons.


    d. `LoadBalancer` service is an extension of the `NodePort` service, which also make possible to call services outside but using some cloud's 
    builtin functionalities. So it's looks like a glue between a cloud platform and the kubernetes cluster. And the main difference that the service
    has an external IP address which could be used for accessing the cluster outside.

    ```
    kind: Service
    apiVersion: v1
    metadata:
    name: bar-service
    spec:
        type: LoadBalancer <-----
    ports:
        # Default port used by the image
        - port: 3000 <---internal port in the cluster
        - targetPort: 3001 <-- where to redirect incoming requests
        - nodePort: 3008 <--- external port number     
    ```

    PS: On the prod usually people use either `Ingress` or the `LoadBalancer` for accessing the cluster.

3. Deployments manage `POD` (like how many pod should be run) etc. So it's better way for starting POD because you have more
power managing them, in comparison when we describe just POD like: 

```
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
  - name: foo-app
    image: kicbase/echo-server:1.0
```

Using this way we cannot set a replica count for instance. In addition if you want delete a deployment with all associated POD then 
you can do it easily. Also using deployments we can inject config and secrets into containers.


4. `Kubelet` a worker node, it's like a service which manages pods and interact between container and node.

5. `Master` node, manages all the running Nodes, and decide where and how many pods should be run in a concrete `Node`, The master node
has an API which you can communicate with using CLI commands. The decision where to tun a POD depends how overload Nodes in a cluster. And the master
nodes selects less overloaded one. Also the master node watch for run POD's statuses, and restart them in case of a failure. In a high load
projects we could use several `master` nodes which are managed by a load balancer.

6. Using namespaces we can logically group resources by their responsibility to avoid accidental rewriting some of them, for instance two
teams may introduce a new deployment with the same name and overwrite  some of existing ones. And also we can limit some resources (CPU, RAM, Storage)
per namespace. 

7. `Helm` it's a package manager for kuber, like yarn, composer, etc. The main idea here that people build their own yaml configuration with appropriate properties and docker images and publish them as packages. Like for example we could search for a package `helm search repo mongoDb`
What else can the `Helm` do:
    a. Release management - you can install any package into the cluster or even rollback it in case of a failure. (Helm keeps all the changes history)
    but it's deleted from the helm version 3.
    b. it also can generate a current cluster configuration which could be lately used in a different cluster, like you could copy the `prod` env into the `dev` one.

8. A `POD` it's like an abstraction layer (isolated virtual host) the smallest unit in the cluster which has it's own IP address and a running a docker container inside it

9. `Persistance` volume it's like a cluster resource and it's like an interface (or like an external plugin to your cluster) where you just describe how much space are you going to use etc.
But the actual storage you should organize your self. We cannot use persistance storages in namespaces because they should be available globally.

10. Persistance volume claim it's just an abstraction like a service where your claim some kind of a storage, and the Persistance volume claim tries
to find an appropriate storage for you and you will be working with a matched one. Claims should be placed in the same namespace as a `POD`.

11. `Storage class` it's kind of a factory which creates storages dynamically when you claim them. It's very convenient way in comparison with the manual
mode.

12. In kuber we can use both `configMap` and `Secret` as a key value storage, for configuring our app. We can even provide files inside the cluster
using. Well the difference between config map and the secret, that the secret keeps data encrypted, they looks like:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  db_host: mongodb-service
 
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  username: ss
  password: bbb

```

13. Then we need to create a secret to be able to fetch images from the private repos

```
kubectl create secret --namespace=news-api docker-registry news-api-docker-registry \
--docker-server=https://index.docker.io/v1/ \
--docker-username=esase \
--docker-password=dev6alexander

```

and then we can use the secret name for auth:

```
spec:
      serviceAccountName: internal-app
      containers:
        - name: $SERVICE_NAME
          image: $IMAGE_NAME
          imagePullPolicy: IfNotPresent
          env:
            - name: SERVER_PORT
              value: "$PORT"
      imagePullSecrets:
        - name: docker-registry <-----
```

