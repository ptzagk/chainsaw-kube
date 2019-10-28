[ WARNING ] This is a small project that I've created to try some libraries and "stuff", for study purposes, don't use in production without review and configure all properly. 

Why chainsaw? Nothing special, was a github suggestion. =)

Here you'll find:

* Spring-boot application with spring-cloud-config-client, actuator, google jib (docker img creator), oauth2 client.
* Spring cloud config server.
* Helm chart for the applications
* Keycloak installed with helm charts
* Gloo Api Gateway installed with helm charts; routes creation;  

What I'm planning to add

* Add Prometheus for monitoring
* Add Gloo Ingress controller
* Add Gloo Knative 
* Add a Quarkus application
* Support for openshift
* Keycloak cluster

Follow the recipe and should work, otherwise drop an issue:

* openJDK 8
* Maven
* VirtualBox

* Minikube

   motivation: basic kubernetes env. to run applications locally

  install:
  ```
  # download and config
  curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
  sudo install minikube /usr/local/bin
  
  # start a new kubernetes cluster with minikube
  minikube start --cpus 2 --memory 8192

  # first steps
  kubectl config use minikube

  # config to use internal docker environment
  eval $(minikube docker-env)

  ```

* Helm charts

   motivation: easy to deploy containers and stateful services into kubernetes and also have a good catalog of ready-to-use charts and configurations.

   install:

   ```
   # download installation script
   curl -L https://git.io/get_helm.sh | bash

   # configure minikube to use helm as application management
   helm init
   ```

* Keycloak (auth server)

   motivation: provides a great management interface, openID support and it has a great development and support community

   ```
   # configure external repository
   helm repo add codecentric https://codecentric.github.io/helm-charts

   # execute installation on minikube
   helm install --name keycloak --set keycloak.password=admin,keycloak.username=admin codecentric/keycloak
   ```

   Run after keycloak is up and running

   ```
   # observe and wait until keycloak is up - Ctrl + C to leave
   kubectl get pods -w

   # expose keycloak throught a node port
   kubectl expose pod keycloak-0 --name=keycloak --type=NodePort

   # check IP:PORT to access the admin console
   minikube service keycloak --url
   ```

   IMPORTANT:

   There's some step to configure on keycloak:

   1. From the "Realm Setting" > "Keys" click on the public key button and copy the public key to be used on the chainsaw-config-server
      as is configured on this Github repo: https://github.com/brunojensen/chainsaw-kube-config
   2. Create a new client called "bank-platform" with "Client Protocol"="openid connect".


   > Optionally, we also could install keycloak-gatekeeper to work as a side-car to app pod and provide authentication out-of-shelf

* Gloo (api gateway)

   motivation: easy-to-use and configure and integrates very well with kubernetes.
   It also expect /swagger.json path to autoconfigure
   the available REST APIs and add them to the api gateway route.

   ```
   # add new repo
   helm repo add gloo https://storage.googleapis.com/solo-public-helm

   # install gloo and setup to use a NodePort
   helm install gloo/gloo --set gatewayProxies.gateway-proxy.service.type=NodePort --name gloo

   # check everything in your minikube
   kubectl get all --all-namespaces

   # get all the external links
   minikube service list

   # install gloo command line tool, required to configure the api gateway later
   curl -sL https://run.solo.io/gloo/install | sh
   export PATH=$HOME/.gloo/bin:$PATH
   ```

* Service discovery comes off-the-shelf with kubernetes

   motivation: kubernetes provides a way to identify services and balance the request between the multiple instances of the application.

* Monitoring with spring actuator and kubernetes liveness/readness probe

   motivation: as a quick solution to monitor the health of the application and configure the environment to restore in case of failure

* Spring-boot Services

   motivation: spring-boot provides an easy-to-go template generation for microservices. One good alternative was to use the quarkus project, but I didn't want to go that far and the requirements of this code challenge is to use spring. But to create the docker image I decide to use a google container tool called jib.

   ```
   # pre-defined bash script to run maven and helm to package and install the application on the local environment
   chmod +x app.sh && ./app.sh install

   # configure api-gateway proxy
   chmod +x k8s-config/gloo-routes && ./k8s-config/gloo-routes
   ```

* Application evaluation

   ```
   # requires "jq" -> https://stedolan.github.io/jq/download/
   # use this script to run tests on the APIs.
   ./app.sh

   ```
