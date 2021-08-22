# Instructions to setup Kubernetes Server with K3s on Digitalocean or any vps provider

This is a guide on how to install K3s Kubernetes cluster with a vps provider. It includes how to properly install traefik as a gateway. The Traefik Kubernetes Ingress provider is a Kubernetes Ingress controller. it manages access to cluster services by supporting the Ingress specification. You use it to connect your pods to the internet.

## Tools Used

- kubectl - [Link](https://kubernetes.io/docs/tasks/tools/)

- kubectx - A utility to manage and switch between kubectl(1) contexts.

    [Windows](https://github.com/thomasliddledba/kubectxwin),  

    [Mac or Linux](https://github.com/ahmetb/kubectx)

## 1. Register on digitalocean and setup vps

Click on the button below for 60 day $100 free credit for new users. It is my affiliate link

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=0a229afd221d&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## 2. Setup K3sup cli tool

This is needed to be installed on your computer. It will handle most of the k3s setup.

K3sup Github [Link](https://github.com/alexellis/k3sup).

## 3. K3s install on ubuntu server

- Install k3s with k3sup tool

    ```bash
    export IP=YOUR SERVER IP ADDRESS HERE

    k3sup install --ip $IP --user root --k3s-extra-args '--no-deploy traefik'
    ```

## 4. Install Traefik

- Create a new namespace for traefik
  
    ```bash
     kubectl create namespace traefik
    ```

- Install 01-crd.yaml, 02-deployment.yaml, 03-loadbalancer.yam from github source folder traefik

     ```bash
     cd traefik

     kubectl apply -f 01-crd.yaml  --namespace=traefik

     kubectl apply -f 02-deployment.yaml  --namespace=traefik

     kubectl apply -f 03-loadbalancer.yaml  --namespace=traefik
    ```

    or if your in the root folder

    ```bash
     kubectl apply -f traefik/  --namespace=traefik
    ```

- check deployment
  
  ```bash
    kubectl get pods --namespace=traefik
    kubectl port-forward svc/traefik-ingress-service 8080
  ```

- what you should see when you type localhost:8080 in your browser
