# RKE2

## Description

RKE2, also known as RKE Government, is Rancher's next-generation Kubernetes distribution. It is a fully conformant Kubernetes distribution that focuses on security and compliance within the U.S. Federal Government sector.

## Directory

- [RKE2](#rke2)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Creating a Kubernetes Cluster](#creating-a-kubernetes-cluster)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Reverse Proxy](#reverse-proxy)
    - [Description](#description-2)
    - [References](#references-2)
    - [Reverse Proxy Networking Setup](#reverse-proxy-networking-setup)
    - [Reverse Proxy Application Setup](#reverse-proxy-application-setup)

## References

- [RKE2](https://docs.rke2.io)

---

## Creating a Kubernetes Cluster

### Description

This details the process of creating and setting up a Kubernetes cluster based on RKE2.

### References

- [Orked](https://github.com/irfanhakim-as/orked)
- [Quick Start](https://docs.rke2.io/install/quickstart)

### Steps

1. [Provision the required number of nodes](../courses/vm.md#creating-a-virtual-machine-from-a-template) as the Master node, Worker node, and **optionally**, the Login node:

   - Please refer to Orked's [prerequisites](https://github.com/irfanhakim-as/orked/blob/master/README.md#prerequisites) and [hardware requirements](https://github.com/irfanhakim-as/orked/blob/master/README.md#hardware-requirements) for the exact details.

   - It is highly recommended that you [create a VM template](../courses/vm.md#creating-a-virtual-machine-template) out of each of these categories of nodes for ease of reproducibility and scaling in the future.

2. Follow the detailed steps laid out in the Orked [installation guide](https://github.com/irfanhakim-as/orked/blob/master/README.md#installation) to setup a RKE2-based Kubernetes cluster from the ground up using a collection of scripts.

---

## Reverse Proxy

### Description

This details how to set up a reverse proxy on the [Kubernetes cluster](#creating-a-kubernetes-cluster) for applications and services.

### References

- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress)
- [Service](https://kubernetes.io/docs/concepts/services-networking/service)

### Reverse Proxy Networking Setup

[Setting up a domain name](#) and [creating subdomain(s)](#) for your service(s) and application(s) only routes external traffic to your homelab environment, not to your service. To route this incoming traffic to your service(s) appropriately:

1. Assuming a reverse proxy such as [Ingress NGINX Controller](https://github.com/irfanhakim-as/orked#ingress-nginx) has been deployed and set up on your [Kubernetes cluster](#creating-a-kubernetes-cluster), identify and take note of its external (i.e. local) IP address, HTTP port, and HTTPS port:

    ```sh
    kubectl --context <cluster> --namespace ingress-nginx get svc ingress-nginx-controller -o jsonpath='External IP: {.status.loadBalancer.ingress[0].ip}{"\n"}{range .spec.ports[*]}{.name}: {.port}{"\n"}{end}'
    ```

    For example:

    ```sh
    kubectl --context my-cluster --namespace ingress-nginx get svc ingress-nginx-controller -o jsonpath='IP address: {.status.loadBalancer.ingress[0].ip}{"\n"}{range .spec.ports[*]}{.name}: {.port}{"\n"}{end}'
    ```

    Sample output:

    ```sh
      IP address: 192.168.0.100
      http: 80
      https: 443
    ```

2. Domain name(s) you have or will set up for your application(s) are expected to be pointed to the public IP address of your homelab environment (i.e. `203.0.113.0`) - this will route external traffic to your home network, but not to your service(s). To address this, set up the following [port forwarding](../courses/network.md#port-forwarding) rules on your router to send traffic to the reverse proxy (i.e. Ingress NGINX Controller):

   - HTTP:

     - Service name: Set a unique, descriptive name for the HTTP port forwarding rule (i.e. `my-cluster-http`)
     - Device IP address: Add the external IP address of the reverse proxy service (i.e. `192.168.0.100`)
     - External port: Add the [default port](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports) for HTTP (i.e. `80`)
     - Internal port: Add the HTTP port of the reverse proxy service (i.e. `80`)
     - Protocol: Set the protocol to `UDP`
     - Enabled: Ensure the port forwarding rule is active by enabling the corresponding checkbox or switch

   - HTTPS:

     - Service name: Set a unique, descriptive name for the HTTPS port forwarding rule (i.e. `my-cluster-https`)
     - Device IP address: Add the external IP address of the reverse proxy service (i.e. `192.168.0.100`)
     - External port: Add the [default port](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports) for HTTPS (i.e. `443`)
     - Internal port: Add the HTTPS port of the reverse proxy service (i.e. `443`)
     - Protocol: Set the protocol to `UDP`
     - Enabled: Ensure the port forwarding rule is active by enabling the corresponding checkbox or switch

    These port forwarding rules ensure that external requests can reach the reverse proxy controller inside the cluster.

3. After completing this network setup, the following [application setup](#reverse-proxy-application-setup) is required for each application(s) and service(s), so the reverse proxy could route external traffic to them accordingly.

### Reverse Proxy Application Setup

This details how to set up reverse proxy for an application or service, in order for traffic reaching the reverse proxy controller could be routed to the application or service accordingly:

1. After setting up the [reverse proxy networking environment](#reverse-proxy-networking-setup) in your homelab, ensure that you have [set up a domain name](../courses/network.md#setting-up-a-domain), and [created a subdomain name](../courses/network.md#registering-subdomains) for your application, if you have not already.

2. At this point, external traffic to your domain(s) will be routed to your home network (i.e. `203.0.113.0`), then to the reverse proxy controller (i.e. `192.168.0.100`) - for the traffic to reach your application or service, the reverse proxy controller needs to know how to route the traffic of a certain domain or subdomain to the correct (Kubernetes) service. This is done by deploying an Ingress resource for said service:

   - An [Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress) helps define rules that map incoming requests to the appropriate [Kubernetes service](https://kubernetes.io/docs/concepts/services-networking/service) - directing users to the service(s) you wish to serve publicly:

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        annotations:
          cert-manager.io/cluster-issuer: letsencrypt-dns-prod
          cert-manager.io/private-key-algorithm: ECDSA
          nginx.ingress.kubernetes.io/affinity: cookie
          nginx.ingress.kubernetes.io/affinity-mode: persistent
          nginx.ingress.kubernetes.io/proxy-body-size: 100m
          nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
          nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"
          nginx.ingress.kubernetes.io/session-cookie-name: route
          nginx.org/client-max-body-size: 100m
        name: <ingress-name>
      spec:
        ingressClassName: nginx
        rules:
        - host: <domain>
          http:
            paths:
            - backend:
                service:
                  name: <service-name>
                  port:
                    name: <service-http-port-name>
              path: /
              pathType: Prefix
        tls:
        - hosts:
          - <domain>
          secretName: <cert-name>
      ```

      Example Ingress manifest (i.e. `ingress.yaml`) for a particular service:

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        annotations:
          cert-manager.io/cluster-issuer: letsencrypt-dns-prod
          cert-manager.io/private-key-algorithm: ECDSA
          nginx.ingress.kubernetes.io/affinity: cookie
          nginx.ingress.kubernetes.io/affinity-mode: persistent
          nginx.ingress.kubernetes.io/proxy-body-size: 100m
          nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
          nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"
          nginx.ingress.kubernetes.io/session-cookie-name: route
          nginx.org/client-max-body-size: 100m
        name: my-app-ingress
      spec:
        ingressClassName: nginx
        rules:
        - host: my-app.example.com
          http:
            paths:
            - backend:
                service:
                  name: my-app-svc
                  port:
                    name: http
              path: /
              pathType: Prefix
        tls:
        - hosts:
          - my-app.example.com
          secretName: my-app-tls-cert
      ```

      The key configuration in this Ingress is the `cert-manager.io/cluster-issuer` annotation, which should be set to `letsencrypt-dns-prod` - this tells [Cert-Manager](https://github.com/irfanhakim-as/orked#cert-manager) to automatically generate SSL/TLS certificates from Let's Encrypt using DNS validation, ensuring your services are securely accessible via HTTPS.

   - Ingress resource(s) relies on a [Kubernetes service](https://kubernetes.io/docs/concepts/services-networking/service) specific to your application (which should be deployed beforehand) that it is mapped to:

      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: <service-name>
      spec:
        type: ClusterIP
        ports:
        - port: <port-number>
          targetPort: <container-port-name>
          protocol: TCP
          name: <port-name>
        selector:
          app: <deployment-name>
      ```

      Example Service manifest (i.e. `service.yaml`) for a particular application:

      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: my-app-svc
      spec:
        type: ClusterIP
        ports:
        - port: 80
          targetPort: http
          protocol: TCP
          name: http
        selector:
          app: my-app
      ```

   - Most [Helm charts](../courses/kubernetes.md#helm) should already provide both the Kubernetes service and Ingress resource that can be enabled and configured easily as part of the deployment of your application.

3. It is also possible to use the same setup to route external traffic to a service or application that is hosted outside of your Kubernetes cluster, in the same network, with slight changes to our configuration and some additional steps:

   - Since the application was not deployed inside of your Kubernetes cluster, deploy the Kubernetes service without specifying the `selector`:

      ```diff
        apiVersion: v1
        kind: Service
        metadata:
          name: <service-name>
        spec:
          ...
      -   selector:
      -     app: <deployment-name>
      ```

      and since there is no corresponding container we could refer its (target) port to by name, set the `targetPort` to the actual port number of the external application:

      ```diff
        apiVersion: v1
        kind: Service
        metadata:
          name: <service-name>
        spec:
          ...
          ports:
          - port: <port-number>
      -     targetPort: <container-port-name>
      +     targetPort: <external-service-port>
            protocol: TCP
            name: <port-name>
      ```

      Example Service manifest (i.e. `service.yaml`) for an external application:

      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: my-app-svc
      spec:
        type: ClusterIP
        ports:
        - port: 80
          targetPort: 8080
          protocol: TCP
          name: http
      ```

   - Since there's no (way to define a) pod selector in the service, manually deploy an [Endpoints](https://kubernetes.io/docs/concepts/services-networking/service/#endpoints) object to define where traffic should go to reach your external application:

      ```yaml
      apiVersion: v1
      kind: Endpoints
      metadata:
        name: <service-name>
      subsets:
      - addresses:
        - ip: <external-service-ip>
        ports:
        - name: <port-name>
          port: <external-service-port>
          protocol: TCP
      ```

      Example Endpoints manifest (i.e. `endpoints.yaml`) for the external service:

      ```yaml
      apiVersion: v1
      kind: Endpoints
      metadata:
        name: my-app-svc
      subsets:
      - addresses:
        - ip: 192.168.0.106
        ports:
        - name: http
          port: 8080
          protocol: TCP
      ```

   - **Alternatively**, instead of writing and deploying these resources individually, you could utilise and [deploy](helm.md#install-or-upgrade-a-helm-chart) the [`mika/external-svc`](https://github.com/irfanhakim-as/charts/tree/master/mika/external-svc) Helm chart which can be easily configured to create the required resources for you.

4. After completing this setup for the application, local or external to your Kubernetes cluster, you should now be able to reach the application or service using its domain name (i.e. `my-app.example.com`) that has been defined in the Ingress resource.
