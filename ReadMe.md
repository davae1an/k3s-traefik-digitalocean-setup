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

![Traefik Dashboard](/screenshots/traefik-dashboard.PNG "Traefik Dash")

## 5. Install cert-manager for (Certificate Management)

  ```bash
  kubectl create namespace cert-manager

  kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml --namespace=cert-manager
  ```

## 6. Add Domain to Digitalocean

Here is a comprehensive guide on how to setup the nameservers for your domain. It is need so you control it with digitalocean

https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars

Follow these instructions after youve setup name server to add domain

https://docs.digitalocean.com/products/networking/dns/how-to/add-domains/

**NOTE!!** DNS server update may take up to 48 hours to take effect so give it some time. It can even happen a little faster. Just be paitent.
## 7. DNS Wildcard certificates (Lets Encrypt)

In order to get wildcard certificates we have to use the kubernetes ingress to fulfil a challenge. In this setup we try to get a wildcard certificate via a DNS challenge. With a wildcard you can use one certificate for all your subdomains ex. domain.com, blog.domain.com, sale.domain.com will all use same certificate. This is much better than doing a challenge for every subdomain.

- Get DigitalOcean Access Token
  
  ![token1](/screenshots/access-token.png "token")

  ![token1](/screenshots/access-token2.png "token2")

- Convert token to base64 
  
  using git bash or linux mac

  ```bash
  echo  'tokenhere' | base64
  ```

  or use this website

  https://www.base64encode.org/

  paste token in box and press encode button

- Go in the do-dns-issure folder and paste the token in base64 format where '************YOUR DIGITALOCEAN ACCESS TOKEN HERE************'
  
  ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
    name: digitalocean-dns
    data:
    access-token: '************YOUR DIGITALOCEAN ACCESS TOKEN HERE************'

  ```

- Apply Secret to namespace

    create a namespace for the secret where your going to host the webapp. Bellow I named it 'mesmore' (From the root directory)

    ```bash
    cd do-dns-issuer
    
    kubectl create namespace mesmore

    kubectl apply -f 001-do-secret.yaml --namespace=mesmore

    ```
  
- Add your email to the Issuer

  ```yaml
   apiVersion: cert-manager.io/v1
    kind: Issuer
    metadata:
    name: letsencrypt-do-dns
    spec:
    acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        email: '************YOUR EMAIL HERE**************'
        privateKeySecretRef:
        name: letsencrypt-do-dns
        solvers:
        - dns01:
            digitalocean:
                tokenSecretRef:
                name: digitalocean-dns
                key: access-token

  ```

- Apply 002-dns-issuer to same namespace

  ```bash
  kubectl apply -f 002-dns-issuer.yaml --namespace=mesmore
  ```

- Modify 003-wildcard-cert.yaml by replacing both 'domainhere.com' and '*.domainhere.com'
  
  ```yaml
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
    name: web-wildcard-certificate
    spec:
    secretName: web-wild-tls
    issuerRef:
        name: letsencrypt-do-dns
    dnsNames:
        - domainhere.com    <------------ HERE to yoursite.com
        - '*.domainhere.com' <----------- AND HERE to '*.yousite.com'

  ```

- Test Certificate with a webapp

Replace domain names 'DOMAINHERE.COM' with the domain you used for the lets encrypt challenge. Then.

```bash
    kubectl apply -f 004-testwhoami.yaml --namespace=mesmore
```

You should see a website that has a secure connection as bellow

