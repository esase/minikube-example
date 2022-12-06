INSTALLATION
---------------

Install these tools, to start using clusters:

    a. minikube
    b. docker desktop with enabled kuber

USAGE
-----

1. Run the cluster locally: `minikube start`
2. To stop the cluster: `minikube stop`
3. Deletes a local Kubernetes cluster. This command deletes the VM, and removes all associated files.`minikube delete`
4. To run a web dashboard: `minikube dashboard`
5. Returns a local ingress' ip: `minikube ip`

RUNNING
-------

1. to be able to forward traffic from local machine to the cluster - `minikube tunnel`

ADVANCED:
--------

1. To add ingress integration: `minikube addons enable ingress`
2. Addons list: `minikube addons list`
3. Apply a new config to the kuber cluster: `kubectl apply -f ingress.config.yaml`
4. Get ingress public IP: `kubectl get ingress`

LINKS
-----

1. https://medium.com/@javatechie/kubernetes-tutorial-install-run-minikube-in-mac-os-k8s-cluster-369b25b0c3f0
2. https://minikube.sigs.k8s.io/docs/start/
3. https://stackoverflow.com/questions/58561682/minikube-with-ingress-example-not-working