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

RUNNING
-------

1. to be able to forward traffic from local machine to the cluster - `minikube tunnel` (and your ingress resources would be available at "127.0.0.1")

ADVANCED:
--------

2. Addons list: `minikube addons list`
3. Apply a new config to the kuber cluster: `kubectl apply -f ingress.config.yaml`
4. Get ingress public IP: `kubectl get ingress`
5. Get pods IP addresses: `kubectl get pod -o wide`
6. Get linked services with pods: `kubectl get endpoints`
7. Get a list of running services: `kubectl get svc`
8. Get list of system running pods: ` kubectl get pod -n kube-system`
9. Get a list of running services and pods: `kubectl get all`

LINKS
-----

1. https://medium.com/@javatechie/kubernetes-tutorial-install-run-minikube-in-mac-os-k8s-cluster-369b25b0c3f0
2. https://minikube.sigs.k8s.io/docs/start/
3. https://stackoverflow.com/questions/58561682/minikube-with-ingress-example-not-working


NOTES:

1. All communication between `PODS` which are like abstract containers, with dynamic IP addresses, which could be deleted and restored many many times
on Nodes (which are physical computers) performed by using services. Which act like a glue between PODS, services know where PODS are located and how
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