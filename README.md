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
29. Export components by a namespace `kubectl get namespace,deployment,secret,service,ingress -n news-api -o yaml > news-api.yaml`
30. List of installed helm packages `helm list -A`
31. Port forwarding `kubectl port-forward "pod-name" 8080`

RABBITMQ
-----------

1. Installing (it's not working on MAC m1 processors :( ):

`helm repo add rabbitmq-repo https://charts.bitnami.com/bitnami`
`helm install rabbitmq rabbitmq-repo/rabbitmq --namespace rabbitmq-system --create-namespace`

2. Get credentials:

```
Credentials:
    echo "Username      : admin"
    echo "Password      : $(kubectl get secret --namespace rabbitmq-system rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 -d)"
    echo "ErLang Cookie : $(kubectl get secret --namespace rabbitmq-system rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 -d)"
```

3. Accessing the UI `port-forward --namespace rabbitmq-system svc/rabbitmq 5672:5672`


MONITORING CLUSTER:
------------------

For monitoring cluster the best choice now days is the `Prometheus`. The tool gives a rich set of functionalities,
like you can monitor CPU and memory consumptions by your microservices. Additionally we can setup
alert rules, and get notifications by email, slack, etc when something wrong. The monitoring data could be 
visualized by the Grafana tool.

1. Installing 
  `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
  `helm repo update`
  `helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring-system -f alertmanager.yml -f pod-auto-detect.yaml --create-namespace`

2. Updating configs

```
  helm upgrade \
    --namespace monitoring \
    -f prom-custom-values.yaml \
    prometheus prometheus-community/kube-prometheus-stack
```

2. Accessing to the Grafana's UI `kubectl port-forward "prometheus-grafana-7bdd794646-7zscz" 3000 -n monitoring` 
  user: `admin`
  pass: `prom-operator`

3. Uninstalling: `helm uninstall prometheus -n monitoring`

4. Accessing to the Prometheus's UI `kubectl port-forward services/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring`

5. Removing the package `helm uninstall prometheus -n monitoring`

6. Show configs `kubectl exec -it statefulset/$name -n $ns -c prometheus -- cat /etc/prometheus/config_out/prometheus.env.yaml`

7. Most popular queries:
  a. Number of containers by cluster and namespace without CPU limits `count by (namespace)(sum by (namespace,pod,container)(kube_pod_container_info{container!=""}) unless sum by (namespace,pod,container)(kube_pod_container_resource_limits{resource="cpu"}))`

  b. Pod restarts by namespace `sum by (namespace)(changes(kube_pod_status_ready{condition="true"}[5m]))`
  c. Memory consumption `container_memory_working_set_bytes{namespace=~".*-api"}`

8. You can download a predefined list of dashboards from: https://grafana.com/grafana/dashboards/6417-kubernetes-cluster-prometheus/
https://grafana.com/grafana/dashboards/7249-kubernetes-cluster/

or you can build queries manually https://sysdig.es/blog/prometheus-query-examples/

9. A list of a top alerts which could be run in a cluster - https://awesome-prometheus-alerts.grep.to/rules.html#kubernetes

PS: prometheus has shipped with a lot of predefined alerts!!! we only need to select of them.


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

large on demand
```
eksctl create cluster \
--name test-cluster \
--region eu-central-1 \
--nodegroup-name linux-nodes \
--node-type a1.medium
--nodes 2
```

4. Change the number of nodes in the cluster  `eksctl scale nodegroup --region eu-central-1 --cluster=test-cluster --nodes=5 --name=linux-nodes --nodes-min=2 --nodes-max=30`

5. Deleting cluster `eksctl delete cluster --region eu-central-1 --name test-cluster`

6. Installing ingress 

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx-system --create-namespace
```

7. Pushing docker images
  a.To be logged in ECR `aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 663781650777.dkr.ecr.eu-central-1.amazonaws.com`
  b. Describe repositories `aws ecr describe-repositories --region eu-central-1`
  c. Tag the image to push to your repository. `docker tag news-api:1.1 663781650777.dkr.ecr.eu-central-1.amazonaws.com/news-api:1.1`
  d. Get list of images `aws ecr describe-images --region eu-central-1 --repository-name news-api`
  e. Push an image `docker push 663781650777.dkr.ecr.eu-central-1.amazonaws.com/news-api:1.1`
  f. Pul an image `docker pull 663781650777.dkr.ecr.eu-central-1.amazonaws.com/news-api:1.0`
  g. Create a repo 
  `aws ecr create-repository \
    --repository-name hello-repository \
    --image-scanning-configuration scanOnPush=true \
    --region region`
  h. Create a repo if it's missing `aws ecr describe-repositories --repository-names news-api  --region eu-central-1 || aws ecr create-repository --repository-name news-api  --region eu-central-1`

RUNNING ON GOOGLE GLOUD
-----------------------

1. Download and configure skd - https://cloud.google.com/sdk/docs/install
2. Create a new custer 
```
gcloud container clusters create test-cluster \
  --zone=europe-west2 \
  --enable-ip-alias \
  --num-nodes=1
```

3. Delete a cluster
```
gcloud container clusters delete test-cluster --zone=europe-west2
```

4. Installing ingress https://cloud.google.com/community/tutorials/nginx-ingress-gke

DOCKER
-----

1. Run `docker run -p 8080:8080 --rm -it esase/news-api:1.0`

2. build and publish `docker build -f ./infra/docker/Dockerfile -t esase/news-api:1.11 .`
  `docker push esase/news-api:1.11`

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
11. https://stackoverflow.com/questions/73398714/
12. docker-fails-when-building-on-m1-macs-exec-usr-local-bin-docker-entrypoint-sh
13. https://medium.com/google-cloud/migrating-applications-between-kubernetes-clusters-8455cf1bfccd
14. https://jhooq.com/get-yaml-for-deployed-kubernetes-resources/
15. https://medium.com/nontechcompany/cool-kubernetes-command-line-plugins-4b0e50362426
16. https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-pull-ecr-image.html
17. https://medium.com/geekculture/how-to-locally-pull-docker-image-from-aws-ecr-ebebbb4c100
18. https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html
19. https://www.basefactor.com/hello-docker-travis-ci-cd
20. https://stackoverflow.com/questions/37267916/how-to-run-aws-configure-in-a-travis-deploy-script
21. https://devapo.io/how-to-set-up-prometheus-on-kubernetes-with-helm-charts/
22. https://grafana.com/grafana/dashboards/3063-memcached-pods-monitoring-via-prometheus/
23. https://sysdig.com/blog/prometheus-query-examples/
24. https://inostudio.com/blog/articles-devops/nastroyka-kube-prometheus-stack/
25. https://www.rabbitmq.com/kubernetes/operator/install-operator.html
26. https://devtron.ai/blog/creating-production-grade-kubernetes-eks-cluster-eksctl/

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
--docker-username=USER \
--docker-password=PASS

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

14. `StatefullSet` pods deployments have some differences, like:
every pod has a constant id if the pod is dying the new restored pod will have the same id. 
Data should be stored outside the docker container, to not lose data when the cluster or pods
are destroyed. The good example is to run `mysql` cluster like master slaves in the kuber.
Kuber does not provide any mechanism to sync data between pods, etc. We should implement it our selves. because kuber is not perfect for statefull apps (because of that side effects)

15. The `kubelet` is the primary "node agent" that runs on each node. It can register the node with the apiserver using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.